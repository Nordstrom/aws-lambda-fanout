# Slack Command to Add Student Streams

Allows students in the workshop to hook and unhook their streams from
 the fan-out Lambda via Slack slash commands.

In the workshops we run, the student's ID is restricted to a four character alpha-numeric string (e.g. `3dq1`; also
referred to as `stage`) and names match `${stage}Stream` and `${stage}StreamWriter` for
their kinesis stream and writer role respectively. See notes at the end of this readme
on how to adjust these restrictions.

To add a student stream to the fan-out:

```
/hook-stream arn:aws:iam::123456789012:role/0a1bStreamWriter arn:aws:kinesis:us-west-2:123456789012:stream/0a1bStream
```

then, after the workshop is complete, _**before**_ deleting the stream:

```
/unhook-stream arn:aws:kinesis:us-west-2:123456789012:stream/0a1bStream
```

## How to configure and deploy

Requirements are similar to the workshop, i.e. you'll need Node JS and Serverless framework to get this
 working. Obviously, you'll also need appropriate access to [Slack API](https://api.slack.com/) and your Slack account.

### Step 1: Create your Application in Slack API

Login to the [Slack API](https://api.slack.com/) site with the credentials you use to connect to your Slack account.
 Click **Your Apps -> Create New App** and give your app a name and pick your team.

On the **Basic Information** screen of your app, under **App Credentials** copy the **Verification Token** value.
 You will need to put this value in your `private.yml` file as a shared secret between the Slack application and
 your service. You may regenerate this token at any time: just update your `private.yml` and redeploy.

### Step 2: Clone this repo and edit your private info

Make a local clone of this repository, and in the root directory edit the existing `private.yml`
 and add your AWS Account Number and Slack Verification Token from Step 1:

```yaml
AwsAccountNumber: 987654321012
SlackVerificationToken: 3hGpDB9YjgtrdXgMqeKLoCH9
```

### Step 3: Deploy service and get command endpoints

With these values updated, and the proper AWS credentials configured for the command line, use Serverless to
  deploy the Slackbot app with the `deploy` command. You may also want to include the optional stage and region arguments:

```bash
serverless deploy -s myAutoBot -r us-west-2
```

Once the deployment is successful, visit the API Gateway service in the AWS Console. You need to visit
 each of the endpoints for the *stage*. Look under **API Gateway -> _{stage}_-hello-retail-slackbot -> Stages**

Click on the **POST** for each of the two endpoints (`hookstream` and `unhookstream`) then make note of
 the **Invoke URL** values for each. Should be similar to:

```
https://blahxxblah.execute-api.us-west-2.amazonaws.com/myAutoBot/slack/hookstream
https://blahxxblah.execute-api.us-west-2.amazonaws.com/myAutoBot/slack/unhookstream
```

### Step 4: Define slash commands in Slack

Finally, we just need to create the commands in your Slack app and link those commands to the API endpoints.

On the **Basic Information** screen of your app in the [Slack API](https://api.slack.com/) site, click **Features -> Slash Commands**, where you'll enter
 the following two commands:

---
Command: `/hook-stream`

Request URL: `https://blahxxblah.execute-api.us-west-2.amazonaws.com/myAutoBot/slack/hookstream`

Short Description: `Adds a stream to the Hello-Retail workshop.`

Usage Hint: `[writer role ARN] [stream kinesis ARN]`

---
Command: `/unhook-stream`

Request URL: `https://blahxxblah.execute-api.us-west-2.amazonaws.com/myAutoBot/slack/unhookstream`

Short Description: `Removes a stream from the Hello-Retail workshop.`

Usage Hint: `[stream kinesis ARN]`

---
Once those commands are Saved, the commands should now work as expected in Slack.

# Notes

## How to change student AWS resource name restrictions

Currently the student streams and roles are restricted to _exactly four alphanumeric_ character long custom IDs,
 `????Stream` and `????StreamWriter`. So `a907Stream` and `a907StreamWriter` are examples of acceptable
  names.

Please edit the RegEx definitions at the top of `handler.js` to alter these restrictions.
