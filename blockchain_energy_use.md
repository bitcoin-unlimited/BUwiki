# Blockchain Energy Use: A Media Package

Recently Bitcoin's energy use has risen to new heights, causing significant media coverage.  This coverage is universally terrible -- full of incorrect facts, misinformation, and (dare I say it) lies.  It is important to be accurate with information,  both for your journalistic integrity and for your employer since incorrect information could have an adverse effect on millions of people's investments.

I am the founder and lead developer of Bitcoin Unlimited, an organization that has produced Bitcoin mining ("full node") software and currently produces the same for Bitcoin Cash.  I also teach graduate and senior level courses on Bitcoin and blockchains at Umass Amherst.

The following statements are entirely true but may surprise or even shock you.  If you don't understand them, you need to read this entire article before printing misinformation about Bitcoin's and other cryptocurrencies' energy use:

1. If Bitcoin's transaction rate increased by 10 times, or even 100x, or 100000x, its energy use would not increase.
2. If more efficient Bitcoin mining devices are created, Bitcoin's energy use will *stay the same*.
3. If Bitcoin's price doubled, its energy use would slowly rise until it has doubled.
4. If Bitcoin's price halved, its energy use would rapidly halve.
5. Given a stable price, Bitcoin's energy use will halve every 4 years.

And here's a classic piece of misinformation:

6. Although Bitcoin mining uses a lot of renewables, that doesn't matter because it means coal and oil is being burnt elsewhere to power other stuff.


## Bitcoin Energy Use is Defined by Price, not Production

This is the single most important idea to understand.  Whenever Bitcoin commits transactions to the blockchain (by solving a block, which is what the miners do and where all the energy goes), the miner gains transaction fees (negligible) and some free Bitcoins (inflation).  Currently the inflation is about 900 BTC per day, or 6.25 BTC per block.  At current prices, this is 312,500 USD per block!  It costs a few thousand dollars to buy a mining machine.  So holy shit right?!  Buy mining machines and run them!  And that's what many people did.  So now you are competing with everyone else who are running mining machines to be the lucky person to solve a block, which happens about every 10 minutes no matter how many machines are running.  

All those machines cost almost nothing for upkeep compared to the energy they use.  So based on natural economic laws, people will keep turning on mining machines until the aggregate energy cost (energy used by every mining machine in the network) is about 312,500 USD per 10 minute period or 1.875M USD per hour, at today's price of about 50000 USD per BTC.  But at that point mining for bitcoins will cost more in energy than it produces in BTC, so new machines will not be added.  If the price of a bitcoin goes up, more machines are turned on.  If the price of a bitcoin goes down, machines are turned off.

Now I think you understand my statements 3 and 4, with the added observation that it takes time to bring new machines online but they can be turned off pretty much instantly.  And a bit of thinking will allow you to understand 2: basically, if machines became twice as efficient, people will run more machines, until again the energy consumed basically equals the money received.

Statement 5 simply follows from one additional fact:  Bitcoin's inflation rate halves every 4 years.  So 4 years from now miners will receive 3.125 BTC per block, or only 156K USD if the price of BTC doesn't change.  This means they'll have to shut off machines to save energy, because you can't spend 312K USD to produce 156K USD for very long.  Eventually, 100+ years from now, this inflation will dwindle to nothing.  And somewhere between now and then the transaction fees will start to matter, but that is a topic for another time.

## Now Understand What Will Be Known As "The World's Dumbest Decision"

As I described the situation above, I only talked about blocks, not transactions.  That's because creating blocks with 10, 1000, 1000000, or 10000000 transactions in them is irrelevant with respect to mining.  All those transactions get boiled down into a single 80 byte chunk of data before being passed to the miners using an efficient algorithm that can take place on a normal computer (so negligible energy cost), but you'll have to take my course to understand the details.

So taking the Bitcoin energy use and dividing it by the number of transactions is like taking the entire wind energy in the Atlantic Ocean, dividing it by the number of sailboats on the water, and thereby concluding that sailboats consume lots of energy.

But there's a catch.  In "The World's Dumbest Decision", powerful members of the Bitcoin community chose to limit the block size to about 1MB every 10 minutes (note that the average web page is > 2MB today and load in 10ths of a second), and therefore the number of transactions (sailboats) to about 2000 per block.  This split the community almost in half (you might realize at this point what side of that split I was on), and created tremendous opportunities for and profit-driving sidechains and nascent alternative cryptocurrencies, which some developers and miners had significant financial interest in.

So due to the "law of unintended consequences", or what we could also call the "law of either being too stupid to think things through or too selfish to do the right thing", the energy per transaction division actually does makes sense for Bitcoin right now.  

But does not make sense for the industry as a whole, and can be remedied in Bitcoin as soon as its politics change.  So it ought to be presented as such -- a Bitcoin-only and possibly temporary problem.

From this description, I hope you understand #1 now.  If Bitcoin ever increases its transaction volume, it will do so without increasing its energy use.  And if it does not increase its transaction volume, other cryptocurrencies such as Bitcoin Cash, have already done so and so will take a larger and larger slice of Bitcoin's transaction volume until Bitcoin becomes irrelevant.

## Bitcoin's Interaction With Renewable Energy

Its important to understand that energy supply and therefore price varies dramatically from location to location, and from one moment to another.  This is because transporting energy is expensive, and can only occur over existing wires so has inflexible capacity.  If you doubt this, you only need to read about the significant power outage in Texas last week (Feb, 2021), and the huge energy prices that resulted, and also about the success of the Hornsdale Power Reserve (https://www.popularmechanics.com/science/a31350880/elon-musk-battery-farm/).

So statement 6 is simply not true, irrespective of Bitcoin.  Consumption of renewable energy at some location *absolutely does not imply production of non-renewable energy elsewhere*.  You simply can't get that energy from here to there.

And if you look at the location of Bitcoin mining hardware, you will find that much if it is near renewable sources, typically hydro.  This should be your first hint that there is something much more interesting going on than the knee-jerk idea that Bitcoin is evil because it "wastes" power.  

What is actually happening is that hydro plants often produce a huge surplus of energy since the energy they produce is proportional to the water flowing through the dam, not the demand for that energy.  This energy can't be transported far because the grid can't handle the capacity, and no one wants it anyway.  So these hydro plants offer this energy very inexpensively so long as consumption is right near the dam.  Its pretty hard to move an entire factory with all its workers 100s or 1000s of miles away up into some remote mountainous location.  Its pretty easy to move bitcoin miners.

But yes, there's an ugly truth that if the price of bitcoin increases too much too quickly, then it becomes (at least for a while) profitable to run the miners on more expensive, non-surplus energy.  This is what we see during Bitcoin's short "boom" cycles which ironically is the only time you reporters come running over here to write articles.

### Forward Looking Ideas About Energy

Dismissing Bitcoin as a dirty technology may be throwing out an amazing technique to actually aid renewable use, so I'd like to look into my crystal ball a bit and propose some "crazy" ideas.

First, Bitcoin mining could allow greater use of nuclear power, which is generally agreed to be cleaner than coal/gas and new designs seem to be much safer (I'm not here to advocate for nuclear, I'm just here to point out a possible interaction with Bitcoin mining).  The problem with nuclear is that it provides a steady "base load" but can't ramp up and down quickly.  And you know, you can't over produce energy without bringing down the grid.  You have to meet demand exactly (sometimes bulk energy prices actually go negative -- providers literally will PAY YOU to consume energy).  But what if there was a "magic black box" that could soak up extra supply and turn it directly into money?  In that case, nuclear baseline load could be higher.

Second, Bitcoin mining could do the same thing for renewables.  The problem with renewables is their erratic production.  But what if you could profitably over-build renewable sites, so you are producing (say) 150% of power demand on a sunny or windy day?  And then on a cloudy or light wind day you drop down to 100% demand.  Why is this good?  Well, if you built out to produce 100% only on the brightest or windiest days, then on cloudy or just breezy days you have to fire up the coal or gas generator to meet supply.  We cannot use that energy for something else.  Could we build a factory that uses that extra-but-variable 50% to make something like aluminum?  No, because it and its workers would sit idle too often, need to be fired up at strange times-of-the-night, work multi-day-and-night shifts, and sometimes we need to start 1% of that factory's production, other times 100%.  But Bitcoin miners have none of these problems.  So we may be able to deploy Bitcoin miners to convert extra renewable electricity production into money that essentially pays for the deployment of extra renewable capacity.

Finally, a Bitcoin miner is a great device for "load shedding" (this is when the power company automatically turns off power to stop the entire grid from failing).  

You could even imagine it behaving as a lossless energy "teleporter".  Imagine a future where miners are more or less evenly distributed across a geographic region, producing money from solar power.  Does one region need more energy and one region have too much?  Rather than pass the power through lossy, expensive-to-maintain, and "nimby" high voltage power lines, turn off miners in the region that needs energy, and turn them on in the region that has too much.

Co-locating bitcoin mines near surplus hydro is the first step in the future I've outlined here.  And its already happened.  But there are a few reasons these more advanced ideas haven't been fully realized.  The first is simply time.  The second is that the boom cycles of Bitcoin make it profitable to mine (during the boom) with expensive energy.  But as Bitcoin grows, the boom cycles will stabilize (look at prior boom cycles, they were bigger on a percentage basis).

The third is that mining hardware used to be getting iteratively but significantly more efficient on a yearly basis, obsoleting old hardware.  So miners had to run that hardware 24x7 so that it would pay back its investment before becoming obsolete.  However, just like computer CPU performance increases are much smaller now than they were in the 1980's and 1990's, the mining hardware's chips are now approaching state-of-the-art in terms of transistor density.  This happened because its a lot cheaper to produce chips using older technology, so the first mining chips were produced using old and relatively cheap technology.  The next chips used less old and more expensive technology, and so on.  Mining chips literally passed through much the same technologies (generally denoted by the size in nanometers of a single transistor) and therefore transistor density increases as CPUs, but did so in 7 years rather than 40, because all those technologies have already been invented.

So we can expect that the usable life of a bitcoin miner today or in the near future will be long enough to allow them to be powered on intermittently rather than continuously.

Of course, you are thinking that we could just push surplus power into batteries.  For sure, but how much battery capacity is ultimately deployed depends on the continued development of battery technology, their usable life, and the ability to transport that power.  It seems unlikely that a battery will ever be developed that can store a "rainy season's" worth of excess power into a year's worth of demand.  Such a device would look nothing like today's batteries.  It is more likely to efficiently convert electricity into some chemical, like gasoline.  Or perhaps that "storage technology" will be a little black box that converts electricity directly into a new form of money.  This new money is stored easily, cheaply, and without loss basically forever, and can be used to purchase power during the dry season from someone's solar surplus.   

The future of renewable energy production and storage is by no means figured out.  But sitting here today, I think that cryptocurrency mining may play a very interesting role in it.