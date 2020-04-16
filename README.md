# Further
Social Distancing and Contact Interactions

# Abstract
Further is a "social distancing" and "contact interactions" app currently available for iPhone, iPad, and Apple Watch.  In short, the software performs two functions; uses Bluetooth signal strength to determine your proximity to others using the app ("social distancing"), alerting you if you appear too close to others, as well as trading random device identifiers to keep you informed if others you have been in contact with develop COVID-19 symptoms ("contact interactions").  We intentionally refrain from using the term "contact tracing" - the software does not trace, create any identifiable records, or gather any information that could associate these random identifiers with a human being.

# How It Works - Contact Interactions
Upon first launch of the app, your device will generate a random identifier (a UUID) that is locally saved to your iPhone, iPad, or Apple Watch.  If your device(s) are signed into the same iCloud account, all devices will share this identifier, transferred via iCloud Key Value Storage (KVS) and/or the Watch Connectivity framework.  This identifier will remain consistent throughout your experience using the app.  Should you delete the app from all devices, a new identifier will be created the next time 
