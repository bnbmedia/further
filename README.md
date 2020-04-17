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
Further uses the Bluetooth antenna in your iPhone, iPad, or Apple Watch to determine the signal strength between yourself and other users of the app.  When another user is nearby, each device will attempt to "connect" to the other, only performing such task if the Bluetooth service identifier *0xFD6F* is recognized and the advertising name *further_app* is found.  Upon a successful connection, each device will begin calculating the signal strength ("RSSI") of its pair, using a threshold calculation of -60.0db to determine if the paired device is "too close" or "a safe distance" away.  In general testing, a RSSI of **greater than** -60.0db has shown a paired device within 2m range of another, while a RSSI of **less than** -60.0db has shown a paired device further than 2m distance from another.  These calculations have been shown to be impacted by environmental interference, device battery strength, and in some cases, weather conditions.  When a paired device is noted as showing a RSSI of **greater than** -60.0db, the Further app will update its interface (and, where applicable, perform haptic feedback), indicating that another user of the app is likely to be within 2m of oneself.  These calculations are meant to be general and should be used as a guide only; using personal awareness and in-person social distancing markers (as are common in grocery stores, pharmacies, and medical facilties at this time) should be a priority over indicators within the app.

When a paired device "disconnects" (or becomes further than an estiamted 2m distance by RRSI), the Further app will note the *disconnect time*, updating the `disconnectTime` variable with the current Unix timestamp.  Should a device that was previously paired come into range again, the `connectTime` and `disconnectTime` will be updated, but the original record of the interaction will remain saved locally (and to iCloud, where applicable).

# API Calls
The Further app uses the following REST API calls to perform transfer of data between your local device(s) and the Amazon Web Services server hosting data.  The following endpoints are in use;

`https://mlv3dsc5tc.execute-api.us-east-1.amazonaws.com/health` - This endpoint accepts a *JSON* object using the *POST* HTTP method.  The data model, as listed in the above *Questionnaire* section, is the payload of this request.  For full transparency, no authorization or authentication methods are required.

`https://mlv3dsc5tc.execute-api.us-east-1.amazonaws.com/responses` - This endpoint can be called using the *GET* HTTP method.  The returned data is an array matching the aforementioned data model, as listed in the above *Questionnaire* section.  This data is updated every 5 minutes, returning only entries in which the user has reported a positive COVID-19 result (per the `testResult` boolean variable) and the `lastUpdate` time is less than 14 days from the current query.  When called from outside of the app, this endpoint returns the data with rotated unique identifiers (UUIDs) to further privacy and anonyimity.

`https://mlv3dsc5tc.execute-api.us-east-1.amazonaws.com/features` - This endpoint is called at app launch using the *GET* HTTP method.  The returned data is a *JSON* object, indicating the available features of the app (in this case, social distancing, the questionnaire, and updates to interactions).  In the event features are required to be disabled (or re-enabled) for any reason, this call will update the user interface accordingly.

# Data
The full set of gathered data is available by performing a *GET* request on this URL;
`https://mlv3dsc5tc.execute-api.us-east-1.amazonaws.com/responses`

As noted above, a call to this endpoint (or visiting this URL in a browser) will return a JSON array of Questionnaire models.  This data is updated every 5 minutes, providing only responses where a user has self-reported a positive COVID-19 test result within the last 14 days.

# Next Steps
Feedback from many interested parties have provided additional thinking points and feature developments to enhance the stability, security, and usefulness of the Further app.  The following is a running list of development tasks that are under consideration:

[] Require user approval before Questionnaire responses are submitted to the server - This idea was based on the ACLU's review and proposal of Apple/Google's contact tracing protocol, suggesting that user adoption would be higher (and privacy strengthened) if users were required to submit additional approval before their responses to questions left the device.

[] Questionnaire and Interaction Updates on Apple Watch - In its current form, the watchOS variant of the app provides social distancing functionality.  As watchOS 6.0 and higher allow for apps to run independently, additional functionality to bring parity to iOS/iPadOS app should be implemented.

[] Rotating UUID and Public Key Exchange - To further strength privacy and security, the idea of a rotating unique device identifier should be implemented.  Alongside this, calls to the web-based server should require an associated public key, ensuring that calls originate from device only (this would prevent any potential tampering).  By rotating the unique device identifier on a given schedule, and strengthening web calls, it is believed that the commitment to privacy and security would be enhanced.

[] Support for macOS - While trivial to bring the Further app to macOS thanks to the Catalyst technology, the usefulness of the technology on macOS is debated.  As non-essential businesses begin re-opening, having an always-on and fully-functional broadcast point (a "beacon") could allow for businesses to leverage a macOS desktop or notebook, ensuring that all employees of a given location could be made aware of potential interactions with other users who may have reported a positive COVID-19 test.
