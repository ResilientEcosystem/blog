---
layout: article
title: PBFT ViewChange Protocol and CheckPoint Algorithm in NexRes
author: Dakai Kang
tags: NexRes PBFT ViewChange CheckPoint Recovery
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/PbftCommit/agreement.jpg

---

The view-change protocol and checkpoint algorithm provide PBFT with liveness by allowing the system to make progress when the primary fails. In this blog, we introduce how we implement the PBFT view-change protocol and checkpoint algorithm in NexRes, list what Byzantine failures our current version is resilient to, and illustrate how our view-change protocol and checkpoint algorithm provide liveness in the presence of the failures.

We use the same model in the PBFT paper. There are $n = 3f+1$ replicas, at most $f$ of while can be Byzantine, where a Byzantine replica can have arbitrary malicious behavior.

## view-change protocol

### FAILURE DETECTION

In this section, we describe how PBFT in NexRes detects the failure of the primary $P$.

- Client $C$ sends a request $T$ to the primary and sets a timer $t_c$.
```c++
  replica_communicator_->SendMessage(*new_request, GetPrimary());
  AddWaitingResponseRequest(std::move(new_request));
```
- If $C$ receives $f+1$ valid responses for $T$ before $t_c$ expires, then $C$ considers $T$ as stable in the ledger.
```c++
  CollectorResultCode ret =
      AddResponseMsg(context->signature, std::move(request),
                     [&](const Request& request,
                         const TransactionCollector::CollectorDataType*) {
                       response = std::make_unique<Request>(request);
                       return;
                     });
  if (ret == CollectorResultCode::STATE_CHANGED) {
      RemoveWaitingResponseRequest(hash);
  }
```
- If not, $C$ broadcasts $T$ to all replicas.
```c++
    if(CheckTimeOut(client_timeout.hash)){
      auto request = GetTimeOutRequest(client_timeout.hash);
      if(request){
        replica_communicator_->BroadCast(*request);
      }
    }
```
- When receiving $T$, if a replica $R$ is not the current primary, then it forwards to the current primary $P$ and sets a timer $t_r$.
```c++
  if (config_.GetSelfInfo().id() != message_manager_->GetCurrentPrimary()) {
    replica_communicator_->SendMessage(*user_request,
                                  message_manager_->GetCurrentPrimary());
    request_complained_.push(std::make_pair(std::move(context), std::move(user_request)));
    return -3;
  }
```
- When $t_r$ expires, $R$ checks if it has committed any request form $C$ since it starts $t_r$.
- If not, $R$ requests a *view-change*.
```c++
    if(viewchange_timeout->type == ViewChangeTimerType::TYPE_COMPLAINT){
      if (status_ == ViewChangeStatus::NONE && viewchange_timeout->view == system_info_->GetCurrentView()) {
        if (viewchange_timeout->start_time >= message_manager_->GetLastCommittedTime(viewchange_timeout->proxy_id)) {
          checkpoint_manager_->TimeoutHandler();
        }
      }
    } 
```

### REQUESTING A VIEW-CHANGE

- Detecting the failure of the primary is not the only reason to request a view-change. Besides, if a replica $R$ receives f+1 VIEW-CHANGE messages of the current view from other replicas and $R$ has not requested a view-change in the current view, then $R$ requests a view-change as well.
```c++
  size_t request_size = AddRequest(viewchange_message, request->sender_id());
  if (request_size == config_.GetMaxMaliciousReplicaNum() + 1) {
    checkpoint_manager_->TimeoutHandler();
  }
```
- Once a view-change is requested, replicas process no more consensus messages and then proceed to construct the **VIEW-CHANGE** message and broadcast it to all the replicas at periodic intervals.
```c++
  checkpoint_manager_->SetTimeoutHandler([&]() {
    if (status_ == ViewChangeStatus::NONE) {
      view_change_counter_ = 1;
    } else if (status_ == ViewChangeStatus::READY_NEW_VIEW) {
      view_change_counter_++;
    }
    if (ChangeStatue(ViewChangeStatus::READY_VIEW_CHANGE)) {
      SendViewChangeMsg();
    }
  });
```
- The **VIEW-CHANGE** message contains a set $S$ of requests that $R$ has prepared, which means that for each request $T\in S$, $R$ has received $2f+1$ **PREPARE** messages in support of $T$.
- For each $T\in S$, the **VIEW-CHANGE** message includes a corresponding *Prepare-Proof* that contains the $2f+1$ digitally-signed **PREPARE** messages.
```c++
  for (int i = 1; i <= max_seq; ++i) {
    if (message_manager_->GetTransactionState(i) >= TransactionStatue::READY_COMMIT) {
      std::vector<RequestInfo> proof_info =
          message_manager_->GetPreparedProof(i);
      auto txn = view_change_message.add_prepared_msg();
      txn->set_seq(i);
      for (const auto& info : proof_info) {
        auto proof = txn->add_proof();
        *proof->mutable_request() = *info.request;
        *proof->mutable_signature() = info.signature;
      }
    }
  }
```
- $R$ moves to the next view, broadcasts the **VIEW-CHANGE** message, and sets a timer $t_v$. If $R$ fails to receive $2f+1$ **VIEW-CHANGE** messages, then $R$ broadcasts the **VIEW-CHANGE** message for the current view again.
```c++
  if(viewchange_timeout->type == ViewChangeTimerType::TYPE_VIEWCHANGE){
      if(status_ == ViewChangeStatus::READY_VIEW_CHANGE && viewchange_timeout->view == system_info_->GetCurrentView()){
      LOG(ERROR)<< "It is time to rebroadcast viewchange messages";
      ChangeStatue(ViewChangeStatus::VIEW_CHANGE_FAIL);
      checkpoint_manager_->TimeoutHandler();
    }
  }
```


###  PROPOSING NEW-VIEW

- If the replica $R$ receives $2f+1$ **VIEW-CHANGE** messages for the current view, $R$ sets a timer $t_n$.
- If $R$ is the new primary $P'$, then $P'$ constructs and broadcasts a **NEW-VIEW** message.
```c++
  size_t request_size = AddRequest(viewchange_message, request->sender_id());
  if (request_size == config_.GetMinDataReceiveNum()) {
    if (IsNextPrimary(viewchange_message.view_number())) {
      SendNewViewMsg(viewchange_message.view_number());
    }
    else {
      StartNewViewTimer();
    }
  }
  ChangeStatue(ViewChangeStatus::READY_NEW_VIEW);
```
- The **NEW-VIEW** message contains the $2f+1$ digitally-signed **VIEW-CHANGE** messages.
```c++
  auto requests = viewchange_request_[view_number];
  NewViewMessage new_view_message;
  new_view_message.set_view_number(view_number);

  std::map<uint64_t, std::string> new_view_request;  // <sequence, digest>
  for (auto& it : requests) {
    ViewChangeMessage& msg = it.second;
    msg.set_view_number(view_number);
    *new_view_message.add_viewchange_messages() = msg;
  }
```

### STARTING A NEW VIEW

- If $R$ fails to receive a valid **NEW-VIEW** message before $t_n$ expires, then $R$ requests one more view-change, enters the next view, and broadcasts **VIEW-CHANGE** message.
```c++
    if(viewchange_timeout->type == ViewChangeTimerType::TYPE_NEWVIEW){
      if(status_ == ViewChangeStatus::READY_NEW_VIEW && viewchange_timeout->view == system_info_->GetCurrentView()){
        checkpoint_manager_->TimeoutHandler();
      }
    }
```
- If a valid **NEW-VIEW** message with the highest sequence number (round number) $max_s$ is received, then, for each sequence number $r < max_s$ with a valid *Prepare-Proof*, $R$ broadcasts the corresponding **COMMIT** message for round $r$, while for each sequence number $r' < max_s$ without a valid *Prepare-Proof*, $R$ broadcasts **PREPARE** message of *null* value for round $r'$. 
```c++
  for (size_t i = 0; i < request_list.size(); ++i) {
    if (new_view_message.request(i).type() ==
        static_cast<int>(Request::TYPE_PRE_PREPARE)) {
      new_view_message.request(i);
      replica_communicator_->SendMessage(new_view_message.request(i),
                                         config_.GetSelfInfo());
    } else {
      replica_communicator_->BroadCast(new_view_message.request(i));
    }
  }
```

## CHECKPOINT ALGORITHM

We have introduced the view-change protocol above. However, view-change protocol cannot prevent malicious replicas from keeping up to $f$ non-faulty replicas in the dark. Malicious replicas do so by not sending any consensus messages replicas to the $f$ replicas. Thus, the $f$ non-faulty replicas cannot commit any new requests while other non-faulty replicas can.

To solve this problem, we need to implement the checkpoint algorithm.

### SENDING CHECKPOINT

- When initiating the system, we set a checkpoint watermark size value, such as 5. Then, every 5 consecutive rounds, a replica $R$ broadcasts a **CHECKPOINT** message with sequence number $r$, where $r$ is the highest sequence number of the 5 rounds, informing other replicas that it has committed and executed the requests in the last 5 rounds. 
```c++
    if (current_seq == last_ckpt_seq + water_mark) {
      last_ckpt_seq = current_seq;
      BroadcastCheckPoint(last_ckpt_seq, last_hash, stable_hashs, stable_seqs);
    }
```

### FETCHING UNKNOWN REQUESTS

- If a replica $R$ receives $f+1$ valid **CHECKPOINT** messages of the same sequence number $r$ and the highest sequence number that $R$ has executed is $r', r'<r$, then, $R$ needs to fetch the unknown requests behind round $r'$.
- $R$ calls an interface to fetch the requests, asking the senders of the $f+1$ **CHECKPOINT** messages one by one until it fetches valid requests.
- To check the validity of the fetched requests, $R$ verifies the hash and signatures.
- If the fetched requests are valid, then $R$ commits and executes them.
```c++
    auto replicas_ = config_.GetReplicaInfos();
    for(auto &replica_: replicas_){
      auto requests = txn_accessor.GetRequestFromReplica(last_seq_ + 1, committable_seq, replica_);
        if (PassAllChecks(requests)) {
          for (auto& request: *requests) { 
            if (executor_) {
              executor_->Commit(std::make_unique<Request>(request));
            }
          }
          break;
        }
    }
```

### STABLE CHECKPOINT

- If a replica $R$ receives $2f+1$ valid **CHECKPOINT** messages of the same sequence number $r$. It means that at least $f+1$
non-faulty replicas have executed requests with sequence number $r$. And we consider $r$ as a *stable checkpoint*. 
```c++
    if (it.second.size() >=
      static_cast<size_t>(config_.GetMinDataReceiveNum())) {
      stable_seq = it.first.first;
      stable_hash = it.first.second;
    }
      
    std::vector<SignatureInfo> votes;
    if (current_stable_seq_ < stable_seq) {
      votes = sign_ckpt_[std::make_pair(stable_seq, stable_hash)];
      stable_ckpt_.set_seq(stable_seq);
      stable_ckpt_.set_hash(stable_hash);
      stable_ckpt_.mutable_signatures()->Clear();
      for (auto vote : votes) {
        *stable_ckpt_.add_signatures() = vote;
      }
      current_stable_seq_ = stable_seq;
    }
```
- A *Stable-Checkpoint-Proof* with sequence number $r$ can be considered as *Prepare-Proof* for all rounds before $r$ because a request *committed* by a non-faulty replica must have been *prepared* by at least $f+1$ non-faulty replicas. A *Prepare-Proof* for the request will definitely appear in at least one of the $2f+1$ **VIEW-CHANGE** messages in the **NEW-VIEW** message.

### OPTIMIZING VIEW-CHANGE WITH CHECKPOINT

- With the checkpoint algorithm, we can reduce the overhead of the view-change protocol. Instead of all *Prepare-Proofs* from the first round, a **VIEW-CHANGE** message contains a *Stable-Checkpoint-Proof* with the highest sequence number $r$ and all *Prepare-Proofs* with a higher sequence number $r', r'>r$.
```c++
  *view_change_message.mutable_stable_ckpt() =
      checkpoint_manager_->GetStableCheckpointWithVotes();
  for (int i = view_change_message.stable_ckpt().seq() + 1; i <= max_seq; ++i) {
    if (message_manager_->GetTransactionState(i) >= TransactionStatue::READY_COMMIT) {
      std::vector<RequestInfo> proof_info =
          message_manager_->GetPreparedProof(i);
      ...
      ...
    }
  }
```

## RESILIENCE TO BYZANTINE BEHAVIORS

In this section, we list the Byzantine behaviors that our implementation is resilient to and illustrate how our implementation is equipped with resilience. 

### NON-RESPONSIVE PRIMARY

We consider a primary as *non-responsive* if it crashes or deliberately stops broadcasting new **PRE-PREPARE** messages. If the primary becomes *non-responsive*, then the client cannot receive responses and will complain. Thus, replicas will forward the complained requests to the primary and start a timer. If the primary keeps being *non-responsive*, then at least $2f+1$ non-faulty replicas will request a *view-change* and eventually enter a new view.

### KEEPING REPLICAS IN THE DARK

Obviously, non-faulty replicas in the dark can catch up with other replicas via the checkpoint algorithm.

### EQUIVOCATION

There are two cases when the primary equivocates in round $r$, which means that the primary sends different **PRE-PREPARE** messages to different replicas. In the first case, no request gets enough votes to be *committed* by a non-faulty replica, which is the same as the case when the primary becomes *non-responsive*. In the second case, there is one request that gets *committed* by some of the non-faulty replicas, which is the same as the case when the primary attempts to keep some replicas in the dark. From what we have illustrated for the two malicious behaviors above, we know that we are able to recover from both of the cases when the primary equivocates.

### CONSECUTIVE FAULTY PRIMARIES

There can be consecutive Byzantine primaries in consecutive views. Thus, it is possible that the new primary is *non-responsive* as well and does not broadcast **NEW-VIEW** messages. To provide liveness in such a case, as mentioned before, non-faulty replicas will request a new *view-change*.

### DUPLICATION

A byzantine primary or primaries in different views may broadcast **PRE-PREPARE** the same request twice, which we call *duplication*. To eliminate *duplication*, when proposing a new **PRE-PREPARE** message, non-faulty primaries check if the request is already proposed. And non-faulty replicas check if the request is already proposed when receiving a **PRE-PREPARE** message.

```c++
int Commitment::ProcessNewRequest(std::unique_ptr<Context> context,
                                  std::unique_ptr<Request> user_request) {
  ...
  if (duplicate_manager_->CheckAndAddProposed(user_request->hash())) {
    return -2;
  }
  ...                                  
}
```

```c++
int Commitment::ProcessProposeMsg(std::unique_ptr<Context> context,
                                  std::unique_ptr<Request> request) {
    ...
    if (duplicate_manager_->CheckAndAddProposed(request->hash())){
      LOG(INFO) << "The request is already proposed, reject";
      return -2;
    }
    ...
}
```