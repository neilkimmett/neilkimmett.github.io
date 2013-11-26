---
layout: post
title:  "Deploying to Testflight"
date:   2013-11-14 22:38:40
---

Getting your app in front of real users as soon as possible is very important. They will use the app in ways you hadn't even thought of, and uncover hidden bugs and gotchas. My current beta distribution platform of choice is [Testflight](http://www.testflightapp.com). I keep meaning to try [HockeyApp](http://hockeyapp.net), but everyone else here at M&S uses Testflight making it the path of least resistance. Again this is very important as you want it to be as easy as possible for testers to install your app. This is the tale of getting new betas out to testers as painlessly as possible.

The most basic way of uploading builds to Testflight is by creating an archive using Xcode (Product > Archive) then uploading it using the [web uploader][Testflight Uploader]. This is a tedious manual process involving a whole lot of clicking. We can do better.

[Testflight Uploader]: https://testflightapp.com/dashboard/builds/add/

![Testflight web uploader](/assets/testflight1.png)
<figcaption>Using the Testflight web uploader. Grim.</figcaption>

For a while I used [Testflight's Mac menubar app][testflight-mac-app], which intelligently watches for any archives you make using Xcode and then offers to upload them. The app then steps you through the process of selecting a provisioning profile, writing release notes, and finally uploading. This removes some friction from the process, but again involves a lot of clicking and hassle. Again, we can do better.

[testflight-mac-app]: https://testflightapp.com/desktop/

![Testflight Menu bar app](/assets/testflight2.png)

<figcaption>Using the Testflight Mac app. Slightly less grim.</figcaption>

In steps the prolific [Matt Thompson](http://www.twitter.com/mattt) who has written an excellent collection of command line tools for automating various aspects of iOS development and distribution called [Nomad](http://www.nomad-cli.com). The specific tool we want is called [Shenzhen](https://github.com/nomad/shenzhen), and allows you to build archives on the command line and distribute them to Testflight (and HockeyApp and FTP). After a quick `gem install shenzhen`, we can simply

{% highlight bash %}
cd <project directory>
ipa build
ipa distribute
{% endhighlight %}

(Or we could do `ipa distribute:testflight`, but `distribute` defaults to Testflight)

As someone who loves the command line this fills me with delight, no more dicking around filling out forms and clicking on boxes, just straight terminal goodness. But wait! You guessed it, we can do better.

Rather than typing (well, copypasting) in our Testflight API token and team token each time, we can feed them to `ipa` using the `--team-token` and `--API-token` flags. To do so we'll wrap up `ipa` in our own little bash script that contains our Testflight credentials, like so<sup>1</sup>

{% highlight bash %}

API_TOKEN="<your api token>"
TEAM_TOKEN="<your team token>"
ipa build
ipa distribute --api_token $API_TOKEN \
               --team_token $TEAM_TOKEN

{% endhighlight %}

You can find your API token and your team token [here](https://testflightapp.com/account/#api) and [here](https://testflightapp.com/dashboard/team/edit/) respectively. Note we could use the flags `-a` and `-T` for the API and team tokens respectively, but I prefer to use the full versions in scripts for readability purposes.

Frequently when I come to write releases notes for new build I forget a lot of the new things in that particular build. To help this I like to write release notes as I go, popping them in a `releasenotes.txt` file. Conveniently `ipa` includes a `--notes` option, which you can pass release notes into. Lets update our script, taking care to deal with the case where we don't have any release notes.

{% highlight bash %}

API_TOKEN="<your api token, found at >"
TEAM_TOKEN="<your team token, found at >"
NOTES="releasenotes.txt"

ipa build
if [ -f $NOTES ];
then
   ipa distribute --api_token $API_TOKEN \
                  --team_token $TEAM_TOKEN \
                  --notes "`cat $NOTES`"
else
   ipa distribute --api_token $API_TOKEN \
                  --team_token $TEAM_TOKEN
fi

{% endhighlight %}

Another pain point for me is accidentally distributing a build with the wrong provisioning profile. We can get our script to help us with this, the simplest way being bailing if the right provisioning profile isn't set in our `project.pbxproj` file.

{% highlight bash %}

API_TOKEN="<your api token>"
TEAM_TOKEN="<your team token>"
NOTES="releasenotes.txt"

if grep --quiet 'PROVISIONING_PROFILE = "< prov prof hash >";' SuperCoolApp.xcodeproj/project.pbxproj
then
  ipa build

  if [ -f $NOTES ];
  then
     ipa distribute --api_token $API_TOKEN \
                    --team_token $TEAM_TOKEN \
                    --notes "`cat $NOTES`"
  else
     ipa distribute --api_token $API_TOKEN \
                    --team_token $TEAM_TOKEN
  fi
fi

{% endhighlight %}
    
Its quick, its dirty, but it works and thats the most important thing. I'd quite like the script to automatically set the correct provisioning profile for me, but thats a challenge for future me.

Finally, we can add a bit of messaging to indicate the script's progress (with added ticks and crosses and colours) and use the `ipa` flags `--lists` and `--notify` to set permissions for the build and notify testers. The final script in all its glory looks like this

{% highlight bash %}

API_TOKEN="<your api token>"
TEAM_TOKEN="<your team token>"
NOTES="releasenotes.txt"


textreset=$(tput sgr0) # reset the foreground colour
red=$(tput setaf 1)
green=$(tput setaf 2)

if grep --quiet 'PROVISIONING_PROFILE = "< prov prof hash >";' SuperCoolApp.xcodeproj/project.pbxproj
then
  echo -e "${green}✔ Correct provisioning profile, yay${textreset}"
  ipa build

  if [ -f $NOTES ];
  then
     echo -e "${green}✔ Using release notes from ${NOTES}${textreset}"
     ipa distribute --api_token $API_TOKEN \
                    --team_token $TEAM_TOKEN \
                    --lists Testers \
                    --notify \
                    --notes "`cat $NOTES`"
  else
     ipa distribute --api_token $API_TOKEN \
                    --team_token $TEAM_TOKEN \
                    --lists Testers \
                    --notify
  fi
else
  echo -e "${red}✘ Incorrect provisioning profile, change it plz${textreset}"
fi

{% endhighlight %}

Hope this helps you get betas of your app out there quicker and I'd love to hear any suggestions for improvements. Find me at [@neilkimmett](http://www.twitter.com/neilkimmett) or email me at [neil@kimmett.me](mailto:neil@kimmett.me).

## Update

It turns out theres a much nicer way of specifying a provisioning profile to use. `xcodebuild` (which `ipa` uses under the hood) uses a `PROVISIONING_PROFILE` environment variable, which we can add to the top of our script and get rid of that messy `grep` business. Much nicer:

{% highlight bash %}

API_TOKEN="<your api token>"
TEAM_TOKEN="<your team token>"
NOTES="releasenotes.txt"
PROVISIONING_PROFILE="< prov prof hash >"

textreset=$(tput sgr0) # reset the foreground colour
red=$(tput setaf 1)
green=$(tput setaf 2)

ipa build # this now uses PROVISIONING_PROFILE

if [ -f $NOTES ];
then
   echo -e "${green}✔ Using release notes from ${NOTES}${textreset}"
   ipa distribute --api_token $API_TOKEN \
                  --team_token $TEAM_TOKEN \
                  --lists Testers \
                  --notify \
                  --notes "`cat $NOTES`"
else
   ipa distribute --api_token $API_TOKEN \
                  --team_token $TEAM_TOKEN \
                  --lists Testers \
                  --notify
fi

{% endhighlight %}

<p></p>

## Update 2

So it turns out that the wrong provisioning profile was sometimes still sneaking its way into the build do. Fear not, I have a new tactic to stop this: a bit of `sed` magic. `sed` is a standard Unix tool for doing find-and-replace in a file. What we need to find is a line like `PROVISIONING_PROFILE = "stuff";` in our `project.pbxproj` file, and replace it with `PROVISIONING_PROFILE = "our actual provisioning profile";`. We can do this with this incantation:

{% highlight bash %}

sed -n -E "s/PROVISIONING_PROFILE = \".+\";/PROVISIONING_PROFILE = \"actual prov prof\";/g" App.xcodeproj/project.pbxproj

{% endhighlight %}

This uses [regular expressions][regex]. The `-E` flag tells OS X's `sed` to use the extended regular expression syntax. The`-n` flag suppresses output from `sed` (otherwise it would spit out the entire contents of your `project.pbxproj` file). The bit after the first `/` is what we're looking for, and the bit after the second `/` is what to replace it with. The `.` in our pattern means "match any character" (except a line break). The `*` means "match the previous character zero or more times". Combined `.*` matches our provisioning profile hash. We can slot this `sed` wizardry into the rest of our script, and we should finally have the One True Release script:

[regex]: http://en.wikipedia.org/wiki/Regular_expression


{% highlight bash %}

API_TOKEN="<your api token>"
TEAM_TOKEN="<your team token>"
NOTES="releasenotes.txt"
PROVISIONING_PROFILE="< prov prof hash >"

textreset=$(tput sgr0) # reset the foreground colour
red=$(tput setaf 1)
green=$(tput setaf 2)

sed -n -E "s/PROVISIONING_PROFILE = \".+\";/PROVISIONING_PROFILE = \"$PROVISIONING_PROFILE\";/g" App.xcodeproj/project.pbxproj

ipa build

if [ -f $NOTES ];
then
   echo -e "${green}✔ Using release notes from ${NOTES}${textreset}"
   ipa distribute --api_token $API_TOKEN \
                  --team_token $TEAM_TOKEN \
                  --lists Testers \
                  --notify \
                  --notes "`cat $NOTES`"
else
   ipa distribute --api_token $API_TOKEN \
                  --team_token $TEAM_TOKEN \
                  --lists Testers \
                  --notify
fi

{% endhighlight %}

<section class="footnotes">
**1.** Depending on your situation (e.g. open source project or not), I would recommend keeping this file out of your version control system to keep others' grubby hands off your Testflight credentials
</section>