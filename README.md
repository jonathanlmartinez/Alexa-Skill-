# Alexa-Skill-
Simple Alexa skill that connects to my capstone using a AWS Lambda function and Node.js 
I will make a blog post and a video soon. This is mainly for Arun.

#Intent Schema

```{
  "intents": [
    {
      "intent": "GetEvents"
    },
    {
      "slots": [
        {
          "name": "Date",
          "type": "AMAZON.DATE"
        }
      ],
      "intent": "GetEventsDate"
    },
    {
      "slots": [
        {
          "name": "Category",
          "type": "LIST_OF_CATEGORIES"
        }
      ],
      "intent": "GetEventsCategory"
    }
  ]
}```

#Custom Slot Types (optional) 

#Type : LIST_OF_CATEGORIES

```Events
Wi-Fi
Parties
USB charging
Parking
Toilets
Water Fountains
Other```


#Sample Utterances

```GetEvents free things around me
GetEvents free things around
GetEvents stuff around me
GetEvents whats around
GetEvents things to do around
GetEvents things to do around me
GetEventsDate free events {Date}
GetEventsDate events {Date}
GetEventsDate things to do {Date}
GetEventsCategory free {Category} around me
GetEventsCategory nearby {Category}```

#Actual Function Code

#Code entry type: Edit code inline
#Runtime: Node.js 4.3
##HandlerInfo: index.handler

```var https = require('https');

exports.handler = (event, context) => {

  try {

    if (event.session.new) {
      // New Session
      console.log("NEW SESSION");
    }

    switch (event.request.type) {

      case "LaunchRequest":
        // Launch Request
        console.log(`LAUNCH REQUEST`);
        context.succeed(
          generateResponse(
            buildSpeechletResponse("Welcome to freester, search events by date or category ", true),
            {}
          )
        );
        break;

      case "IntentRequest":
        // Intent Request
        console.log(`INTENT REQUEST`);

        switch(event.request.intent.name) {
          case "GetEvents":
            var endpoint = "https://freester.herokuapp.com/api/v1/locations.json"; // ENDPOINT GOES HERE
            var body = "";
            https.get(endpoint, (response) => {
              response.on('data', (chunk) => { body += chunk });
              response.on('end', () => {
                var data = JSON.parse(body);
                var title = data.data[0].title;
                var location = data.data[0].location;
                var splitLocation = location.split(',');
                var address  = splitLocation[0];
                
                
                context.succeed(
                  generateResponse(
                    buildSpeechletResponse(`${title} located at ${address}` , true),
                    {}
                  )
                );
              });
            });
            break;


          case "GetEventsDate":
            console.log(event.request.intent.slots.Date.value);
            fecha = event.request.intent.slots.Date.value;
            var endpoint = "https://freester.herokuapp.com/api/v1/locations.json" // ENDPOINT GOES HERE
            var body = ""
            https.get(endpoint, (response) => {
              response.on('data', (chunk) => { body += chunk });
              response.on('end', () => {
                var data = JSON.parse(body);
                function getLocationByDate(date) {
                  return data.data.filter(
                      function(data){ return data.date == date}
                  );
                }
                
                var found = getLocationByDate(fecha);
                
                var title = found[0].title;
                var date = found[0].date;
                var location = found[0].location;
                var splitLocation = location.split(',');
                var address  = splitLocation[0];
                
                context.succeed(
                  generateResponse(
                    buildSpeechletResponse(` found ${title} located at ${address}` , true),
                    {}
                  )
                );
              });
            });
            break;
            
            case "GetEventsCategory":
            console.log(event.request.intent.slots.Category.value);
            category = event.request.intent.slots.Category.value;
            function capitalize(string) {
            return string.charAt(0).toUpperCase() + string.slice(1);
            }
            capitilizedCategory = capitalize(category);

            var endpoint = "https://freester.herokuapp.com/api/v1/locations.json" // ENDPOINT GOES HERE
            var body = ""
            https.get(endpoint, (response) => {
              response.on('data', (chunk) => { body += chunk });
              response.on('end', () => {
                var data = JSON.parse(body);
                function getLocationByCategory(category) {
                  return data.data.filter(
                      function(data){ return data.category == category}
                  );
                }
                
                var found = getLocationByCategory(capitilizedCategory);
                var items = Object.keys(found).length
                var title = found[0].title;
                var date = found[0].date;
                var location = found[0].location;
                var splitLocation = location.split(',');
                var address  = splitLocation[0];
                
                context.succeed(
                  generateResponse(
                    buildSpeechletResponse(` Found ${title} at ${address}` , true),
                    {}
                  )
                );
              });
            });
            break;
            
            
            

          default:
            throw "Invalid intent";
        }

        break;

      case "SessionEndedRequest":
        // Session Ended Request
        console.log(`SESSION ENDED REQUEST`);
        break;

      default:
        context.fail(`INVALID REQUEST TYPE: ${event.request.type}`);

    }

  } catch(error) { context.fail(`Exception: ${error}`) }

};

// Helpers
buildSpeechletResponse = (outputText, shouldEndSession) => {

  return {
    outputSpeech: {
      type: "PlainText",
      text: outputText
    },
    shouldEndSession: shouldEndSession
  };

};

generateResponse = (speechletResponse, sessionAttributes) => {

  return {
    version: "1.0",
    sessionAttributes: sessionAttributes,
    response: speechletResponse
  };

};```
