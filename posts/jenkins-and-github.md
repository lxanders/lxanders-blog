{
  title: 'Jenkins with GitHub integration',
  date: '2014-07-07',
  author: 'Alexander Schmidt'
}

Even now that [Travis CI](https://travis-ci.org/) is having more and more success (one of the main point being its simplicity and ease of use) not everyone has it in his or her technology stack as it is not free for private company use.
One of the great features `Travis` offers is its integration with GitHub. At least this point can be achieved with [Jenkins](http://jenkins-ci.org/) too by installing some plugins and taking some time to configure it appropriately.

The feature you will achieve by following the steps below will be:

- Push code to a GitHub repository and a build is triggered
- Have the result of the build displayed in GitHub (e.g. in a pull request). This is very useful as you will see if everything (seems to be) ok prior to merging the changes back to a stable branch like master

As I found out (twice) the configuration part is not as easy as it could and should be. That's why I decided to write down some of my experiences.

Prepare GitHub
--------------

Register a ssh key for your `Jenkins` server:

- If you don't have a ssh key I highly recommend adding one - just follow the instructions in [the GitHub help](https://help.github.com/articles/generating-ssh-keys). Note: You have to use the public part of your generated key here
- You can use the private part of your key for `Jenkins` later

Configure the web hooks:

- Visit your repository on GitHub and click `Settings`
- Select `Webhooks & Services`
- Click `Add Service -> Jenkins (GitHub plugin)`
- The configuration depends on the fact if your `Jenkins` server is reachable over the internet
  - If yes: You can see the URL under which your `Jenkins` waits for messages from the outside when you go to the general settings and click on the question mark icon next to the `Let Jenkins auto-manage hook URLs` in the `GitHub Web Hook` area. Enter this URL in the GitHub `Jenkins hook url` field
  - If no: Unfortunately this means you have to let `Jenkins` poll for changes. Check the `Poll SCM` option and enter a schedule for it. To check every minute for changes you can use `* * * * *`. This works quite good for pull requests but doesn't for single commits but as I said at least you will have the major advantage of seeing if your branch was building succesful

Create a `personal access token` on your GitHub users `application` settings page:

- Important: Make sure that at least the following checkboxes are selected otherwise you won't have the right permissions
  - `repo`
  - `admin:org`
  - `admin:public_key`
  - `admin:repo_hook`
- Remember this generated token - you will need it later as your `OAuth token` for `Jenkins` (and won't be able to see it again once you leave the page where it was shown)

Prepare Jenkins
---------------

Install the Git and GitHub plugin for `Jenkins`:

- Startup Jenkins
- Go to `Manage Jenkins -> Manage Plugins -> Available`
- Select `Git Plugin` and `GitHub Plugin` (if they are not available you might need to go to the advanced options and manually trigger `Check now`)
- Click `Download now and install after restart` - the download of the packages and dependencies will begin
- Tick the checkbox for restarting the server after this step has finished and the server will restart
- Add a new `New Item` or select one of your already existing jobs and click `Configure`

Configure the `Git Plugin`: 

- In `Source Code Management` select `Git`
  - Point the `Repository URL` to your GitHub repository (e.g. https://github.com/yourUserName/TestRepo)
  - Select the correct credentials for your GitHub repository or add them now
  - If you created a keypair earlier You can use the private key part of it here
  - On a production environment you would most likely have your own ssh keys on the machine that is running `Jenkins` - in this case you would use them
  - As always with such keypairs you have the private key here for which you entered the public key on GitHub)
  - The username you use for the credential is only for finding this credentials in `Jenkins` they are not important for something else
- Clear the content of the `Branches to Build` field - this configures `Jenkins` to build all branches if changed

Configure the `GitHub Plugin`:

- In `Build Triggers` select `Build when a change is pushed to GitHub`    
- In the `Post-build Actions` section click `Add post-build action -> Set build status on GitHub commit`
- Go back to the general configuration for `Jenkins` (the `Manage Jenkins` settings we opened already earlier) and select `Configure System`
- Under `GitHub Web Hook` select `Let Jenkins auto-manage hook URLs`. This enables you to enter credentials etc.
- In `OAuth token` enter the token you created on the GitHub page for your user earlier

Word of warning
---------------

This guide is to get you started on integrating `Jenkins` with GitHub. You absolutely want to get some professional help for making things secure regarding the keys used and how your `Jenkins` server is available from the outside.
