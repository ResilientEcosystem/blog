---
layout: article
title: ResilientDB - Passing Custom Messages
author: alex
tags: Protocol ResilentDB
aside:
    toc: true
---

This is an installment of the ResilientDB tutorial series. ResilientDB is a permissioned blockchain fabric developed by the developers at the Exploratory Systems Lab at UC Davis. We are a group of researchers on a mission led by Prof. Mohammad Sadoghi to pioneer a resilient data platform at scale. 
{:.info}

Message passing forms the basis in distributed computing protocols, and is no different than what is found in ResilientDB. It relies on supporting infrastructure to invoke processes, subroutines, or functions. If you are interested in sending new types of messages from node to node in ResilientDB, then let's get started.

### Defining Your Message

Here will be an example of of new type of message called `SCAN_MSG`.

Each message must be inherited from the class `Message`. We must define the following pure virtual functions:

```c++
class ScanMessage : public Message
{
public:
    void copy_from_buf(char *buf);
    void copy_to_buf(char *buf);
    void copy_from_txn(TxnManager *txn);
    void copy_to_txn(TxnManager *txn);
    uint64_t get_size();
    void init() {}
    void release() {}

    uint64_t return_node;
};
```

Since we have an extra data member in this message (`uint64_t return_node`), the copy to/from buffer routines must accordingly account for this during the serialization process of writing onto the character buffer. 

Here is an example of that:

```c++
void ScanMessage::copy_to_buf(char *buf)
{
	Message::mcopy_to_buf(buf);
	uint64_t ptr = Message::mget_size();
	
	COPY_BUF(buf, return_node, ptr);
}
```

The helper function `COPY_BUF(v, d, p)` defined in `system/helper.h` is simply a wrapper over the `memcpy` routine, where a pointer p designates where to begin writing from one buffer to another, in addition to incrementing `p` by an appropriate size.

Similarly there is a `COPY_VAL` performing the same thing in reverse which will be utilized in the `void ScanMessage::copy_from_buf(char *buf)` member function.



Also you must extend the factory function `Message *Message::create_message(RemReqType rtype)` by adding to the case statement:

```c++
case SCAN_MSG:
    msg = new ScanMessage;
    break;
```


Additionally you need to support the member function of deleting a messages resources in `void Message::release_message(Message *msg)`:

```c++
case SCAN_MSG:
{
    ScanMessage *m_msg = (ScanMessage *)msg;
    m_msg->release();
    delete m_msg;
    break;
}
```

### Message Sending

Before a message can be sent to the message queue, it must be signed upon instantiation. This is done via the `sign()` member function of each respective `Message` type. Additionally there is two vectors that must be enqueued on the `msg_queue`. This is all performed by the message sender:

```c++
vector<string> emptyvec;
emptyvec.push_back(scanMsg->signature);
vector<uint64_t> dest;
dest.push_back(CLIENT_TARGET_ID);
```

For these messages to be able to get enqueued, they must added to the case statement in `void MessageQueue::enqueue`. 

```c++
case SCAN_MSG:
     for (uint64_t i = 0; i < ndsign.size(); i++) {
         ((ScanMessage *)msg)->sign(dest[i]);
         entry->allsign.push_back(ndsign[i]);
     }
     break;
```

That routine specifies the pushing of said message's signatures onto `entry->allsign` of a `msg_entry` as well as the designation of which replicas you intend to pass the message off to.


#### How Signing is Performed 

Messages are signed and verified in their overloaded `sign(uint64_t dest_node)` and `validate()` member functions.

```c++
void ScanMessage::sign(uint64_t dest_node) {
#if USE_CRYPTO
	string message = this->getString();
	signingClientNode(message, this->signature, this->pubKey, dest_node);
#else
	this->signature = "0";
#endif
	this->sigSize = this->signature.size();
	this->keySize = this->pubKey.size();
}
```

For a `Message` to be signable, it must also support stringification via its own `getString()` member function.

The `getString()` usually begins with stringifying the `return_node` node (`g_node_id` originally created said message) and then the same to its data members. In the case of the `ClientQueryBatch`, the data being stringified are all the messages that wrap transactions in a batch. 


When a `WorkerThread` object takes ownership of a message, presumebly via dequeueing a message from the `MessageQueue`, it calls a wrapper function `validate_msg()` over the `validate()` function described above. Validation is done in the message processing step shown below. 

Internally, replicas use `signingNodeNode` helper function in their sign member function and `validateNodeNode` for validation and subsequently returns `true`; client aggregator nodes use `signingClientNode` in their signing and `validateClientNode` for validation and also returns `true`.

These routines depend on a third party Crypo++ library. 

If a message is not signed during the verification step a Segmentation fault results:

```c++
Thread 26 "s_worker" received signal SIGSEGV, Segmentation fault.
[Switching to Thread 0x7fe2590e7700 (LWP 172)]
0x0000000000416a0c in CryptoPP::ed25519PublicKey::ed25519PublicKey (this=0x7fe2590e5190,
    __in_chrg=<optimized out>, __vtt_parm=<optimized out>) at deps/crypto/xed25519.h:618
618     struct ed25519PublicKey : public X509PublicKey
```

### Message Retrieval 

On the recieving end, worker threads retrieve these messages and must support the processing of said routine. This is done by defining a routine to perform exactly what should be done on retrieving this special message.


Therefore you must support the case statement in the `void WorkerThread::process(Message *msg)` member function.

```c++
case SCAN_MSG:
    rc = process_scan_msg(msg);
    break;
```

Heres my special task once a message is recieved:

```c++
RC WorkerThread::process_scan_msg(Message *msg)
{
    cout << "process_scan_msg: " << std::endl;
    ScanMessage *smsg = (ScanMessage *)msg;
    validate_msg(smsg);
    std::unordered_map<std::string, std::string> table = db->ScanTable("KV");
    cout << "Accessing DB: " << table.size() << std::endl;

    return RCOK;
}
```


The validation step in `bool WorkerThread::validate_msg(Message *msg)` must also be extended:

```c++
case SCAN_MSG: 
    if (!((ScanMessage *)msg)->validate())
        {
            assert(0);
        }
    break;
```

We must overload the validate member function of the class we were working with, which also involves overloading the `getString()` function.

```c++
bool ScanMessage::validate() {
#if USE_CRYPTO
	string message = this->getString();

	if (!validateClientNode(message, this->pubKey, this->signature, this->return_node)) {
		std::cout << "Validation Failed." << std::endl;
		assert(0);
		return false;
	}

	std::cout << "Validation Success." << std::endl;
#endif
	return true;
}
```