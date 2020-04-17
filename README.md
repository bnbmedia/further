# Further
Social Distancing and Contact Interactions

# Abstract
Further is a "social distancing" and "contact interactions" app currently available for iPhone, iPad, and Apple Watch.  In short, the software performs two functions; uses Bluetooth signal strength to determine your proximity to others using the app ("social distancing"), alerting you if you appear too close to others, as well as trading random device identifiers to keep you informed if others you have been in contact with develop COVID-19 symptoms ("contact interactions").  We intentionally refrain from using the term "contact tracing" - the software does not trace, create any identifiable records, or gather any information that could associate these random identifiers with a human being.

# How It Works - Contact Interactions
Upon first launch of the app, your device will generate a random identifier (a UUID) that is locally saved to your iPhone, iPad, or Apple Watch.  If your device(s) are signed into the same iCloud account, all devices will share this identifier, transferred via iCloud Key Value Storage (KVS) and/or the Watch Connectivity framework.  This identifier will remain consistent throughout your experience using the app.  Should you delete the app from all devices, a new identifier will be created the next time you download and launch the app.

When the app is running in the foreground and Bluetooth permissions have been allowed, Further will broadcast services using the Bluetooth Low Energy (BLE) protcol.  The service identifier *0xFD6F* will be used, alongside a the advertising name *further_app*.  Upon detection of another device broadcasting the same service identifier and advertising name, each user's app will transfer the unique identifier (UUID) of the other user.  This data will be gathered and stored locally in the following format;

```
{
  personUUID: String, // The unique identifier of the found device
  connectTime: Double, // The time of connection, based on a Unix timestamp
  disconnectTime: Double?, // The time of disconnect, based on a Unix timestamp
  hasReceivedNotification: Bool // A boolean variable to be used if this UUID reports a positive COVID-19 test
}
```

When possible, the app will attempt to "connect" to the alternate device, solely for the sake of establishing a "disconnect time," and allowing the app to calculate length of time two devices were in range of one another.  This is not an implemented feature, but is being built with the intent that future health guidance could indicate that transmission of COVID-19 may require a certain time elapsing.

# How It Works - Questionnaire
Within the app exists the functionality to self-report responses to the following questions;

- Whether you are feeling sick and exhibiting symptoms you *think* may be related to COVID-19
- Whether you have been professionally tested for COVID-19 using a CDC-approved test
- Whether your results of that test were positive

Your responses to these questions, along with your device's unique identifier, are sent to a server for storage.  This data is packaged in JSON format and transported via HTTPS to storage housed on Amazon Web Services.  The data transferred appears in the following format;

```
{
  var id: String // The unique identifier of your device
  var feelingSick: Bool // A boolean value indicating your response to whether you are exhibiting symptoms
  var hasBeenTested: Bool // A boolean value indicating your response as to whether you have been tested
  var testResult: Bool // A boolean value indicating your response as to whether the test result was positive
  var lastUpdate: Double // The Unix timestamp as to the last update of your questions
}
```
This data is stored locally on your device (and/or in iCloud, if your device is signed into an iCloud account) as well as transported using the Combine and URLSession frameworks available in iOS.

# How It Works - Interaction Updates
Once per day, the app will query data saved in storage, filtering for the following criteria;

- A "true" value for the `testResult` boolean variable
- A timestamp within the last 14 days for the `lastUpdate` variable

To minimize server impact, this data is queried every 5 minutes and saved to a JSON file.  The app will download this JSON file locally, rather than perform individual queries on the server.  Upon receipt of said file, the app will determine if any locally saved device identifiers exist on the list.  If so, the app will determine if such identifier (per the `hasBeenNotified` variable) has already been exhibited to the user, and if not, include that UUID in the daily update.

Each daily update provides a visual representation of the "risk level," ranging from low to high.  A low risk level indicates **no** exposure to any users self-reporting a positive COVID-19.  A medium risk level indicates **at least one** exposures to a user self-reporting a positive COVID-19 test.  A high risk level indicaites **three or more** exposures to users self-reporting a positive COVID-19 test.  The results of the daily query are saved locally to the device (and/or in iCloud, if the device is signed into an iCloud account).

# How It Works - Social Distancing
