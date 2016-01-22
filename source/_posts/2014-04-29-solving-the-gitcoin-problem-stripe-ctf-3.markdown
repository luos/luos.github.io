---
layout: post
title: Solving the Gitcoin Problem – Stripe CTF 3
date: 2014-04-29 17:45:54 +0200
comments: true
categories: [en, scala]
---

Stripe, the payment provider, origanised a [CTF](https://stripe.com/blog/ctf3-launch), where you could the basics of distributed systems. One of the tasks was to write a bitcoin-ledger-like implementation based on git. 

If you are not well versed in the git implementation, then you should know for this that git creates an object for every commit. This commit contains a sha hash of the contents you are storing plus the hash of the related information. This information contains  the user, email, and commit message.

In bitcoin mining you always want to generate a hash which is less than some kind of constant. For example if you generate hash "123" and requirements contain "001" then your hash is not accepted as it's not less than the required threshold. 

In this tasks we had to generate an sha hash less than the required threshold which as "000001". If we accept that sha hashes are random then it this task actaully says that we have to generate around 16 million hashes in the time they set us, which was around a few minutes. 

In this  problem we had to write a gitcoin miner and we had an example miner which was really slow. 

First I thought it would be good to use Scala/Akka to pass the Tasks around. My plan was to fill up an Actor system and let it handle the hashing using the sys.process package and some commands from the supplied bash example. For some reason it was a complete fail, the Actor got full and used up all my memory and never done anything, in the end I had to shoot this idea (and the process too :) ). 

After this I tried a simple parallel for and calling 

``` scala
git hash-object -t commit --stdin -w <<< commit-body
```

This has never used more than a couple of percent CPU, I don’t know why, maybe git locks the files in the repo?

After this I got the idea to rewrite the git hash-object part in scala, and in the end it worked out.

If we visit the git objects documentation we can see that a commit contains five things:

``` scala
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
parent some-hash
author Scott Chacon <schacon@gmail.com> 1243040974 -0700 
committer Scott Chacon <schacon@gmail.com> 1243040974 -0700
first commit (some description)
```

The first value is the tree, we can obtain this with “git write-tree”. The second value is the parent of the commit, we can obtain this with “git rev-parse HEAD”. The other things in this were just placeholders and the goal was to change the last part of the commit to get a hash with git hash-object which is lower than “000001xxxxxxxxxxxxxxxx…”. This means that the hash must be lower than the string “000001″. This is difficult because the only way to obtain the correct info to get the needed hash to try as many hash as we could in the shortest time possible.

Now we know whats needed by the git hash-object function, we have to know what is the format used to calculate the hash. Turns out git prepends a “commit (body-length)\0″ string to the commit body, so thats what we have to do and we are good to go.

My code looked something like this:

``` scala

val tree = Seq("git","write-tree").!!.trim
val parent = Seq("git","rev-parse","HEAD").!!.trim
val bode = s"tree ${tree}\n" +
s"parent ${parent}\n" +
s"author CTF user <me@example.com> ${now} +0000\n" +
"""committer CTF user <me@example.com> 1390941203 +0000
Give me a Gitcoin
"""
var eq = 0
val max = 100000000
   for( i <- (0 to max).par) {
       val cbode = bode + i+"\n"
       val fullbode = "commit "+cbode.length+"\0" + cbode
       val hash = digest(fullbode).reverse.padTo(40,"0").reverse.mkString
 
       if (hash < difficulty){
          println("-----------------------")
          println(fullbode)
          println("-----------------------")
          val cmd = "git hash-object -t commit --stdin -w <<< \""+cbode.trim()+"\";git reset --hard \""+hash+"\" < /dev/null; git push origin master;"
          println(cmd)
          println("-----------------------")
          println("HASH: "+hash)
          println("GITHASH: "+ commitHash(cbode))
          //System.exit(0)
      }
 }

```

I imported the scala.sys.process._ methods/packages to easily run commands. You can see the syntax in the first couple of rows. After assembling the body of the commit I started a parallel for to use my all my CPU cores ( 2. gen mobile core i5).

Timing my miner, hashing 100000000 commits:

```
../scala-2.10.3/bin/scala ../Scalaminer.scala 
1344,73s user 20,31s system 355% cpu 6:23,53 total
```

Turns out its approximately 261 kHash/sec.  I don’t know if it counts as fast, it was enough to get a winner commit.

Here is the gist of the code with the remnants of previous tries: https://gist.github.com/luos/f2c8098be3ef0e49cee9

In the end I committed by hand because there was some trouble with the endline character and sometimes it was needed at the end of the commit, sometimes not. Strange.


Last updated: 2016, January. 
