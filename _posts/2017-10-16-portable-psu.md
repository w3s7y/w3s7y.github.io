---
title:  "Creating a 'portapsu' from an old laptop charger and a £25 buck converter"
date:   2017-10-16 19:37:00 +0100
categories: blog
---
Introduction
---
So in one of my previous blogs (okay it's probably been in drafts so long now
it probably will never get published!) but I did create a DC bench power supply
out of an old ATX power supply I scavenged from an old PC I had lying around.
While this PSU has some huge benefits, primarily The ridiculous amounts of power
(20 Amps on the 5V and 12V rails) it does have it's fair share of drawbacks...
* It's **big**
* It's **heavy**
* No over current protection (this has caught me out more than I care to share)
* No constant current mode
* It only has 3.3V, 5V and 12V outputs

As I am a fully mobilised resource (I work anywhere in the UK at short notice)
this means that out of necessity I am generally living out of hotels every week
and lugging a huge power supply around just to do a bit of electronic hacking and
learning in the hotel at night just simply isn't viable. I needed something much
smaller (handheld ideally) power supply.

The Video
---
I take absolutely no credit for finding this, I follow [Dave Jones](https://www.youtube.com/user/EEVblog) on
[YouTube](https://youtube.com) and he recently posted [this video](https://www.youtube.com/watch?v=Cw2AjcczHg4&t=1221s) in which he reviews
[this Chinese made DC/DC buck converter](https://www.amazon.co.uk/gp/product/B01L8VD6MS/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
and finishes saying that while he doubts it really would suffer the output
wattages it boats it _is_ a cracking little power supply for a hobbyist.  Oh and
by the way, if like myself you enjoy electronics and engineering in general I
highly recommend his videos.

That was all the recommendation I needed, I scoured [Amazon](https://amazon.co.uk)
and found an identical one rated for 50V and 5A (bigger than the one
Dave shows in the video, but identical in all other respects).  From what I can recall,
he just used his regular DC bench power supply to power his unit and I was going
to do the same and power it off the 12V rail of my ATX PSU
(it wouldn't help the portability issue but would give me advanced
features like CC/CV modes and over current protection on the bench) but then I saw
the answer staring me right in the face (well it was on the floor actually)...

![The chargers](/assets/img/portapsu/powerpacks.jpg)

Laptop chargers are 240AC in... and _something_ DC output to charge the batteries
so I started to have a look at the ones I had lying around from dead laptops
(who doesn't have a load of these knocking around?)

![Back of power pack](/assets/img/portapsu/specs.jpg)

Bingo! as we can see this little power supply unit will put out a good 20VDC at
3.25A (65W).  I did have a much bigger one (175W) but it was still only 20VDC
(and I kinda use it for charging my laptop so yeah I'll leave that one as is!)

The modding
---
So I got the unit in the post...

![dps5005](/assets/img/portapsu/dps5005.jpg)

Checked it's I/O

![bottom](/assets/img/portapsu/bottom.jpg)

Then got the side cutters out and took it out on the power supply

![The snips](/assets/img/portapsu/snips.jpg)

The cord of the power supply can be determined by looking at the spec on the back
of the unit, on this particular one it was a coaxial cable design with the center
solid core being +ve and the outer shielding being -ve after doing a bit cable stripping
I simply screwed the laptop charger outputs into the input on the unit and then
found some suitable coloured 5A wire and screwed those into the output and finished
it all off by hot gluing the unit to the top of the power supply itself and also
added a plastic cable guide that came out of the salvaged washing machine.

![The final product](/assets/img/portapsu/front.jpg)

And a quick shot of the back

![The final product - Back](/assets/img/portapsu/back.jpg)

And running a 6 Ohm 10W load @ 5VDC

![Load Test](/assets/img/portapsu/lt1.jpg)

Running a bit hot!

![Load Test 2](/assets/img/portapsu/lt2.jpg)

Finally, using it to power my oscilloscope (made in another blog) which is measuring
a PWM output pin on my Arduino.  Note the secondary readout on the multimeter.

![Arduino Setup](/assets/img/portapsu/arduino.jpg)

Just for completeness, code that was running on the Arduino side to produce that particular signal.
```
void setup() {
  // put your setup code here, to run once:
  pinMode(3, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  analogWrite(3, 128);
}
```

Conclusion
---
So there it is! A pocket-sized 20V variable CC/CV DC power supply with short protection,
current limiting, memory pre-sets and a whole raft of other features
for the princely sum of £25 and an old laptop power supply.  
Best of all, it's super portable which means I can now take my more serious
projects (which can't be ran off a 1A/5V USB) with me to keep me entertained in the evenings!  

I hope you enjoyed this quick little blog and as always, feel free to hit me up
on Twitter if you have any questions!
