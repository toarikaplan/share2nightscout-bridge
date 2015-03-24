# share2nightscout-bridge

Fetches data from Dexcom's webservice and puts it in Nightscout.

## Status

* [Releases][releases]
* [![wercker status](https://app.wercker.com/status/1d9a86d110cb9d42c844fa60d084e5c4/m "wercker status")](https://app.wercker.com/project/bykey/1d9a86d110cb9d42c844fa60d084e5c4)

## How to use

#### Prerequisites:
You probably don't need both Azure and Heroku.  Only one is required.

* A working Nightscout web app

### Classic Azure

Quick overview, see the [3 step tutorial][tutorial] for more information with
screenshots.

> Visit the [releases page][releases], download `nightscout-sidecar.zip`, and
> upload to Azure as a web job.  Set the environment variables as Connection Strings.

[releases]: https://github.com/bewest/share2nightscout-bridge/releases
[tutorial]: https://github.com/bewest/share2nightscout-bridge/issues/1


1. Download `nightscout-sidecar.zip` file from the
   [releases page][releases]

1. Find the **Configure** tab to set the required `DEXCOM_PASSWORD` and
   `DEXCOM_ACCOUNT_NAME` variables in Connection Settings area.

1. In your Azure site, Create a new `Continuous` web job
  * upload the `nightscout-sidecar.zip` from the [releases page][releases]

[create-webjobs]: http://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/


### Heroku

[![Deploy to Heroku][heroku-img]][heroku-url]

[heroku-img]: https://www.herokucdn.com/deploy/button.png
[heroku-url]: https://heroku.com/deploy

If you use the Deploy to Heroku button, follow these instructions:
1. Read settings instructions carefully. Especially the instruction for url syntax for `WEBSITE_HOSTNAME`

2. When deployment completes, click on "make your first edit"

3. Dynos - Edit - Move slider from "0" to "1"

4. Click save

5. Wait patiently for 3 minutes and you will see updates in mongo and on your site!

### Environment variables

These are required.

Highly recommend setting these as `Custom` type of
**Connection String** environment variable inside of Azure.

Most people with a pre-existing Nightscout with the API enabled (using
the `API_SECRET`) will only need to add two more custom connection
string variables, `DEXCOM_PASSWORD`, and `DEXCOM_ACCOUNT_NAME`.  The
app uses these details to stay connected, bridging the "data streams"
over the web.  If the API is not already set up using the
`API_SECRET`, then the third variable is also needed, set it to any
long phrase of your choosing, it needs to be longer than 12
characters.

* **DEXCOM_ACCOUNT_NAME** - Dexcom Share user name
* **DEXCOM_PASSWORD** - Dexcom Share password
* **API_SECRET** - (might already be set in Azure) your Nightscout API Secret

#### Features

These environment variables are optional, you can most likely ignore
this section.
These features can be used to emulate gap sync.

* **maxCount** - default: `1`, maximum number of records to process
* **minutes** - default: `1440`, number of minutes to include

This one is completely optional.

* **SHARE_INTERVAL** - default: `60000 * 2.5`, number of ms to wait between
  updates.  Default is `2.5` minutes.

#### Differences betweeen Azure and Heroku

* **NS** - Your fully qualified `https://bar.example.com` endpoint
  with no path.  This is similar to your `/pebble` endpoint, but
  without the `/pebble` part.  It starts with `https://` has your website
  hostname, and then nothing after it.

We now look at **`WEBSITE_HOSTNAME`** environment variable, which is set
automatically by [Azure][azure-environment].

Only one of these environment variables needs to be defined; if using Azure
everything is done automatically.  If using Heroku, we recommend setting
`WEBSITE_HOSTNAME` during the creation step.

[azure-environment]: https://github.com/projectkudu/kudu/wiki/Azure-runtime-environment


## How it works:

Using the Share2 app allows an iphone to push data from a Dexcom
receiver to Dexcom's webservices.  This program fetches a user's data
from Dexcom's servers, and stores it in their own Nightscout server.

By default, this program run as `node index.js`, will loop forever.
[As described by Scott Hanselman][blog-post], one of my many advisors,
this logs in to Dexcom Share as the data publisher.  It re-uses the
token every `5` minutes to fetch the `maxCount` latest glucose records
within the last specified `minutes`.  This information is then sent to
the user's specified Nightscout install, making the data available to
the beloved pebble watch and other equipment owned and operated by the
receiver's owner.  It will continue to re-use the same `sessionID`
until it expires, at which point it should attempt to log in again.
If it can log in again, it will continue to re-use the new token to
fetch data, storing it into Nightscout.

[blog-post]: http://www.hanselman.com/blog/BridgingDexcomShareCGMReceiversAndNightscout.aspx

### What to expect

Shortly after creating the webjob, it should start, and your
Nightscout rig should receive data as usual.

This project is not FDA approved, not recommended for therapy, and not
recommended by [Dexcom][dexcom-eula].

[dexcom-eula]: http://www.dexcom.com/node/5421

#### Output/logs

The output looks something like this when it works:
```
$ env $(cat shansel.env )  node index.js 
Entries [ { sgv: 70,
    date: 1426298639000,
    dateString: '2015-03-14T02:03:59.000Z',
    trend: 4,
    device: 'share2',
    type: 'sgv' } ]
Nightscout upload error null status 200 []

```

#### Azure Account Quotas

We **highly recommend** using this only on the paid Azure  plans.  Using this
program with Azure Free website causes the usage to exceed the free quotas.
Feel free to try it out, but watch out for those quotas.  You can partner with
family and friends to run up to 20 sites on a single plan.  Because you have a
paid plan, the service will have paid performance and support, with better
terms on uptimes.  It may be possible to also host mongo on the same paid
account, in which case it the system will **fail much less often**, because of
the **Basic** support.

See the [billing comments][billing-issue] for more details and
feedback.

[billing-issue]: https://github.com/bewest/share2nightscout-bridge/issues/2

