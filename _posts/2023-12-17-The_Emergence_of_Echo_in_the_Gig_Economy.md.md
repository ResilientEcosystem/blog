## The Emergence of Echo in the Gig Economy

**Group 5: Ashwin Chembu, Scott Wu, Zachary Willson, Susanna Mathew**

### Motivation: The Need for Reliable Identity Verification

The gig economy, with giants like Uber, Lyft, and Doordash, has transformed the employment landscape, offering flexibility and convenience. But with these benefits come challenges, especially in identity verification. The authenticity of contractors is crucial in maintaining operational integrity and user trust. Echo arises as a solution to streamline identity verification, enhancing trust and efficiency in the gig economy.

### Challenges of Current Verification Systems

Present systems face issues like inconsistent processes, technological limitations, and privacy concerns. Scaling these systems is often inefficient, leading to bottlenecks. Echo addresses these problems by offering a universal, technologically advanced, and secure solution.

### Echo's Solution: Facial Recognition and Blockchain Integration

Echo integrates advanced facial recognition and ResilientDB blockchain technology, providing a robust framework for identity verification. Our system leverages the MXFace API for accurate facial recognition and mints NFTs on the blockchain for secure and transparent contractor history.

MXFace API calls: 
```javascript
require("dotenv").config()

const  bodyParser = require('body-parser');

  

var  request = require('request');

var  rp = require('request-promise-native'); // Corrected import

const  _apiUrl = 'https://faceapi.mxface.ai/api/v3/face/';

const  _subscriptionKey = process.env.SUBSCRIPTION_KEY;//change subscription key / Key entered for Zachary Willson's account

  

async  function  base64EncodeFromUrl(url) {

try {

let  imageBuffer = await  rp({ // Using rp instead of requestPromise

url: url,

encoding: null

});

return  imageBuffer.toString('base64');

} catch (error) {

console.error('Error downloading or encoding image:', error);

return  null;

}

}

  
  

var  fs = require('fs');

function  base64Encode(file) {

var  body = fs.readFileSync(file);

return  body.toString('base64');

}

  
  

function  getRequestOption(api, encodedImage) {

var  options = {

url: _apiUrl + api,

method: 'POST',

headers: {

'subscriptionkey': _subscriptionKey,

'Content-Type': 'application/json'

},

json: {

encoded_image: encodedImage

},

rejectUnauthorized: false,

};

return  options

}

  

function  sendRequest(api, encodedImage) {

request(getRequestOption(api, encodedImage), function (error, response) {

if (error) {

console.log(error)

}

else {

console.log("Response /" + api);

if (response.statusCode == 200) {

console.log(response.body);

var  faces = response.body.faces; // Assuming faces is an array within response.body

  

if (Array.isArray(faces)) {

for (var  face  of  faces) {

console.log("Face Quality : " + face.quality);

}

} else  if (typeof  faces === 'object') {

// Handle the case where faces is an object instead of an array

console.log("Face Quality : " + faces.quality);

} else {

console.log("Unexpected structure for faces");

}

} else {

console.log("Error :");

console.log(response.body);

}

}

});

}

  
  
  
  
  
  
  
  
  
  
  
  

const  face = async (req, res) => {

  

const { username, image } = req.body;

console.log(req)

console.log(username)

console.log("image",image)

var  base64SingleFace = await  image;

var  base64MultipleFace = await  base64EncodeFromUrl(`https://ucdavis189fecho.s3.us-west-1.amazonaws.com/${username}.jpg`);

sendRequest("detect", base64MultipleFace)

//console.log("detected")

sendRequest("analytics", base64SingleFace)

//console.log("analytics")

sendRequest("landmark", base64SingleFace)

//console.log("landmark")

  
  

var  optionsFaceCompare = {

url: _apiUrl + 'verify',

method: 'POST',

headers: {

'subscriptionkey': _subscriptionKey,

'Content-Type': 'application/json'

},

json: {

encoded_image1: base64SingleFace,

encoded_image2: base64MultipleFace

},

rejectUnauthorized: false,

};

  

request(optionsFaceCompare, function (error, response) {

console.log("Response /verify");

if (error) {

console.log(error)

}

else {

console.log(response.body)

res.json(response.body);

}

});

  
  

}

  

module.exports = {

face

};
```



### System Design: Simplicity and Security

The user interface of Echo focuses on ease of use, featuring a straightforward landing and login page. The system ensures privacy by keeping personal data like photos and personal information off the blockchain, only using them for identity comparison.

### Technical Implementation: MongoDB, ResilientDB, and React

Echo employs MongoDB for secure data storage, ResilientDB blockchain for verification tokens, and a React-based ledger page for user interaction. This blend of technologies ensures security, transparency, and ease of use.

MongoDB API calls: 
```javascript
const { default: mongoose } = require("mongoose");
const Person = require("../models/personModel");

// get all Persons
const getPersons = async (req, res) => {
  const Persons = await Person.find().sort({ createdAt: -1 });
  res.status(200).json(Persons);
};

// get all Passengers
const getPassengers = async (req, res) => {
  const passengers = await Person.find({ type: "passenger" }).sort({ createdAt: -1 });
  res.status(200).json(passengers);
};

// get all Drivers
const getDrivers = async (req, res) => {
  const drivers = await Person.find({ type: "driver" }).sort({ createdAt: -1 });
  res.status(200).json(drivers);
};

// get a specific Driver by name
const getDriverName = async (req, res) => {
  const { id } = req.params;
  const drivers = await Person.find({ looking: id }).sort({ createdAt: -1 });
  res.status(200).json(drivers);
};

// get a single Person
const getPerson = async (req, res) => {
  const { id } = req.params;
  if (!mongoose.Types.ObjectId.isValid(id)) {
    return res.status(404).json({ error: "No such Person and invalid ID" });
  }
  const person = await Person.findById(id);
  if (!person) {
    return res.status(404).json({ error: "No such Person" });
  }
  res.status(200).json(person);
};

// create a new Person
const createPerson = async (req, res) => {
  const { title, description, image, icon, Person, convos } = req.body;
  let emptyFields = [];
  if (!title) {
    emptyFields.push("title");
  }
  if (!Person || Person.length === 0) {
    emptyFields.push("Person");
  }
  if (emptyFields.length > 0) {
    return res.status(400).json({ error: "Please fill in all fields", emptyFields });
  }
  try {
    const user_id = req.user.id;
    const newPerson = await Person.create({
      title,
      description,
      image,
      icon,
      Person,
      convos,
      user_id
    });
    res.status(200).json(newPerson);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

// delete a Person
const deletePerson = async (req, res) => {
  const { id } = req.params;
  if (!mongoose.Types.ObjectId.isValid(id)) {
    return res.status(404).json({ error: "No such Person and invalid ID" });
  }
  const person = await Person.findByIdAndDelete(id);
  if (!person) {
    return res.status(404).json({ error: "No such Person" });
  }
  res.status(200).json(person);
};

// update a Person
const updatePerson = async (req, res) => {
  const { id } = req.params;
  if (!mongoose.Types.ObjectId.isValid(id)) {
    return res.status(404).json({ error: "No such Person and invalid ID" });
  }
  const existingPerson = await Person.findById(id);
  if (!existingPerson) {
    return res.status(404).json({ error: "No such Person" });
  }
  const updatedPersonData = { ...req.body };
  const updatedPerson = await Person.findByIdAndUpdate(id, updatedPersonData, { new: true });
  res.status(200).json(updatedPerson);
};

module.exports = {
  getPerson,
  getPersons,
  createPerson,
  deletePerson,
  updatePerson,
  getPassengers,
  getDrivers,
  getDriverName
};
```
Python SDK (Flask):
```python
from resdb_driver import Resdb
from datetime import datetime
from resdb_driver.crypto import generate_keypair
from flask import Flask, request, jsonify
from flask_cors import CORS
import json

app = Flask(__name__)
CORS(app)
db_root_url = "http://127.0.0.1:18000"
adminkeys = generate_keypair()

@app.route("/test")
def test():  
    return "hello"

@app.route("/create_key")
def create_key():  
    key = generate_keypair()
    print(key)
    key_dict = {
        "private": key[0],
        "public": key[1]
    }
    key_json = json.dumps(key_dict)
    return key_json

@app.route("/create_token", methods=["POST"])
def create_token():
    service, signer_public_key, signer_private_key, user_public_key = request.json['service'], adminkeys[1], adminkeys[0], request.json['user_public_key']
    utc_time = datetime.utcnow()
    utc_time_str = utc_time.isoformat()
    db = Resdb(db_root_url)
    token_data = {
        "data": {
            "start_time": utc_time_str,
            "service" : service,
        },
    }
    prepared_token_tx = db.transactions.prepare(
        operation="CREATE",
        signers=signer_public_key,
        recipients=[([user_public_key], 1)],
        asset=token_data, 
    )
    fulfilled_token_tx = db.transactions.fulfill(prepared_token_tx, private_keys=signer_private_key)
    db.transactions.send_commit(fulfilled_token_tx)
    response_data = {
        "transaction_id": fulfilled_token_tx["id"],
        "message": "Token created successfully",
    }
    return jsonify(response_data)

@app.route("/retrieve", methods=["POST"])
def get_nft():
    nft_data = db.transactions.retrieve(txid=key)
    if nft_data:
        return jsonify(nft_data)
    else:
        return jsonify({"error": "NFT not found"}), 404 

if __name__ == "__main__":
    app.run(debug=True)
```

This script is a good starting point for a blockchain-based application handling tokens and NFTs. However, ensure that the necessary modules and dependencies are installed and properly configured in your environment.

### Why Choose Echo?

The choice of technologies like ResilientDB and MongoDB is rooted in their ability to offer transparency, security, and scalability. These attributes align with the dynamic needs of the gig economy, ensuring a robust and reliable verification process.

### Evaluation Methodology

Echo integrates the Mxface facial recognition model, accessed via API, for identity verification. This process is central to our solution's effectiveness in the gig economy.

### The Future of Echo

Looking ahead, Echo aims to partner with leading gig delivery companies, enhancing the ledger with proprietary data. Our vision includes incorporating biometric data into the verification process for heightened security and accuracy.

### Conclusions

Echo stands as a comprehensive solution to the challenges in the gig economy, offering a transparent, secure, and efficient system for identity verification. By leveraging blockchain technology and advanced facial recognition, Echo redefines trust and safety standards in this dynamic sector.

### Next Steps: Expanding and Refining Echo

Our journey includes evolving the verification process and forging partnerships to broaden Echo's impact in the gig economy. Staying at the forefront of technological innovation, we aim to refine and advance our system to meet and anticipate the evolving industry needs.

----------

This article was inspired by the work and ideas presented by Group 5 (Ashwin Chembu, Scott Wu, Zachary Willson, Susanna Mathew) in their Echo project. For more information on Echo and its technical aspects, visit the [GitHub repository](https://github.com/ashw24/echoresDB) or view a [demo](https://www.youtube.com/watch?v=8_GzAwVrBy8) of Echo in action.

