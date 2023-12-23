# Decentralized chat system based on ResilientDB
## Project description
### Overview
In today's life, when we try to send a message on most chat software,
the message will first be sent to the central server, 
and then forwarded to the target user by the central server. 
The disadvantage of this is that all data will be captured and stored by the central server, 
which greatly increases the risk of data leakage and leakage of private information. 
Now, we will create a decentralized chat system based on the ResilientDB blockchain. 
This decentralized chat system does not store any personal information, 
and only the sender and recipient can encrypt and decrypt the message during the transmission of the message.

### Key Features
1. **Decentralized Architecture:** Our system avoids the need for a central server. 
All messages are transmitted through ResilientDB blockchain.

2. **Security:** By using the asymmetric encryption algorithm (gpg), 
we can ensure that information cannot be easily cracked during blockchain transmission. 
Only the sender and recipient of the message can use their keys to encrypt and decrypt

3. **Privacy-first Approach:** User data is never stored on any central server. No chat history will be stored.

4. **Open-source:** To ensure utmost transparency and security, our system is fully open-source, allowing community participation and review.

### Potential Use Cases
<!-- TODO -->

### Conclusion
<!-- TODO -->

## Schedule

1. **Oct, 20:**
   1. Finish project proposal
   2. Decide which language to use(Python or C++)
   3. Understand SDK API
2. **Nov, 3:**
   1. Finish the detail process of this project
   2. Start working on backend code
3. **Nov, 17**
   1. Basic GUI finished
   2. Backend coding finished
   3. Improve fault tolerance on the chatting system

4. **Dec, 8**
   1. GUI finished
   2. Finish README.md file

## Overall Idea:
- How messages work: The key idea of this project is about using a custom class called Page. It works as a container to send message nad store chat history. Each page can store 20 messages, and each message have 6 different fields
This project is using kv service of the ResilientDB, and the command line instructions are `set [key] [value]` and `get [key]`.
The `[key]` part constructed with two fields `[page name]` and `[page number]`. The `[page name]` is constructed by sender and receiver's public key(sorted in ASCII order) to ensure that both receiver and sender will have the same page name.
Page name will never change throughout the chatting. On the other hand, `[page number]` starts at 1, and it will increase by 1 everytime the page is full. ResChat will load most recent two pages everytime user start up the client(current page and current page -1)
In this way, all chat history are stored on the chain(ResilientDB) and user can load as many as previous chatting history as he/she wants.

- Encryption: This project uses RSA as our encryption algorithm. As I mentioned in above, each message will have six different fields, they are `[[Receiver's public key, message type, timestamp, message type extension, message1(encrypted by sender's pub key), message2(encrypted by receiver's pub key)], ... ]]`
The most important part in it are `Receiver's public key`, `message1`, and `message2`. Since we are using asymmetric encryption, if we only encrypt message by receiver's public key, sender will no longer decrypt this message by his/her private key.
By encrypting message with both sender and receiver's public key, sender and receiver can use `Receiver's public key` field to decide choose which message(message1 or message2) to decrypt


## Function:
#### [page.py](page.py): This is a custom class serves as store and transfer messages. When user want to send message, system will add this message into current page. If the page is full, the sender of the 21th message will create a new page and put the message into the new page
- `self.msg`: This is a 20 by 6 NumPy array, each page contains 20 messages. Each message has 6 different element: Receiver's public key, message type, timestamp, message type extension, message1(encrypted by sender's pub key), message2(encrypted by receiver's pub key)
- `self.message_count`: This is to count how many messages are currently in this page
- `def is_full`: This function check if this page is full
- `def add_message`: This function will add one message into this page
- `def_tostring`: This function will convert Page class into a string. The reason is for subprocess to pass this page in command line, because subprocess doesn't take <class 'Crypto.PublicKey.RSA.RsaKey'> and <class 'datetime.datetime'>.
- `def from_string`: This function acts like inverse of `to_string` function. It converts a string back to page class.
- `def sort_by_time`: This function will sort current messages in this page by their timestamp.
- `def all_messages`: This function will return a list of all messages current in this page

#### [db.py](db.py): Each user will have a local jason database to store all the friend information. The structure of this database is a dictionary, key is nickname, value is another dictionary which stores the corresponding public key and page number
- `def add_friend`: This function will take user picked nickname and a public key to create a new friend in local database
- `def get_friend`: This function will use nickname to find corresponding friend information from local database, it will return a dictionary
- `def update_page_num`: This function will use nickname to increase corresponding page number by one
- `def get_all_friend`: This function will return a list of all friends' nickname in local database

#### [encryption.py](encryption.py): This chatting system is using RSA algorithm to encrypt and decrypt each message
- `def create_keys`: This function will take a username and a password to create a public key and a private key
- `def encrypt_message`: This function takes a message(byte) and a public key to return an encrypted message string
- `def decrypt_message`: This function will user private key to decrypt a message string
- `def load_public_key`: This function will load public key from public_key.pem file
- `def load_private_key`: This function will load private key from private_key.pem file
- `def public_key_to_string`: This function will convert <class 'Crypto.PublicKey.RSA.RsaKey'> to <class 'str'> for store the public key into local database
- `def string_to_public_key`: This function will convert <class 'str'> back to <class 'Crypto.PublicKey.RSA.RsaKey'>

#### [chain_operation.py](chain_operation.py): This file use Python subprocess package to run command line instruction
- `def send_page`: This function will set a page into ResilientDB
- `def get_page`: This function will get a page from the ResilientDB

#### [message.py](message.py): This file contains the whole process of sending messages(pages)
- `def send_message(message, nickname)`: This function includes everything that to send a message
  1. System will first get corresponding friend information from local database by its nickname
  2. Combine both sender and receiver's public key in ASCII order to construct page name
  3. Get current page from the chain(ResilientDB)
  4. Check this page is full or not
     - If Yes: Create a new page and increase local database's page number by 1
     - If No: Continue
  5. Encrypt user's message and add this encrypted message into this page
  6. Set this page onto the chain(ResilientDB)

- `def get_update(nickname, password)`: This function contains the whole process of receiving message(page)
  1. System will first get corresponding friend information from local database by its nickname
  2. Combine both sender and receiver's public key in ASCII order to construct page name
  3. Since ResChat will load two most recent pages by default, it will have three cases:
     1. Both current page and current page - 1 are empty(There are no chat history between these two users): `return None`
     2. Current page have message(s) in it, but not for current page - 1(The current page number is 1, and there are some messages in it): convert current page from string into Page, and None for current page - 1
     3. Both Current page and current page have messages in it(The current page number is > 1): Convert both current page and current page - 1 from string to Page class
  4. Check if the current page is full or not:
     - If Yes: This means sender might already start using a new page. So, system will try to get current page + 1. If there are message(s) in current page + 1, update the local database
     - If No: Continue
  5. Combine current page and current page - 1 into one NumPy array
  6. Use `Receiver's public key` to decide which message to decrypt and decrypt that message
  7. Add decrypted messages into a list
  8. Return that list




## [Command Line Menu](command_line_menu.py)
1. Start up client:
   1. Check if public_key.pem and private_key.pem exist
        - If Yes: Continue
        - If No: Ask user to input the username and password to generate those two keys
2. Main Menu:
   1. Start Chatting
   2. Add Friend:
      1. Ask user input the nickname and public key: Go to second menu
      2. Check if this nickname is already in use
         - If Yes: Report an error
         - If No: Add this friend information to [local_database](local_friends_list.json)
   3. Quit: Exist this program
3. Second Menu:
   1. Pick a user to start chatting:
      1. System will print out all the nicknames in local database
      2. Ask user to select a receiver to start chat with
        - If user input nickname is in database: Go to third menu
        - If user input nickname is not in database: Report an error
   2. Previous Menu

4. Third Menu:
    1. Send message:
       1. Ask user input
       2. Set this message into ResilientDB
    2. Update: Update the chat history