---
layout: post
title:  "Guesstimate: A Programming Model for Collaborative Distributed Systems"
date:   2018-11-16 00:00:00 -0000
categories: pl
group: pl
---

_This is one post in a series about programming models and languages for distributed computing that I'm writing as part of my [history of distributed programming techniques](https://github.com/cmeiklejohn/PMLDC)._

<h2 id="relevant-reading">Relevant Reading</h2>
<ul>
<li><p><em>Guesstimate: A Programming Model for Collaborative Distributed Systems</em>, Rajan, Rajamani, Yaduvanshi, PLDI 2010 <span class="citation">Rajan, Rajamani, and Yaduvanshi (2010)</span>.</p></li>
</ul>
<h2 id="commentary">Commentary</h2>
<p>As we’ve seen previously, distributed applications have to be shoehorned into either the CP or AP model as outlined by the CAP thereom. This results from the fact that applications that must remain available under partition, and as the CAP theorem states that applications cannot be simultaneously AP and CP, the application has to choose whether to remain consistent or sacrifice consistency for availability when partitions inevitably occur.</p>
<p>Guesstimate is a programming model for collaborative applications that aims to reduce latency by replicating objects on participating nodes in a network. Guesstimate provides the user with an object-oriented programming model: objects can be replicated and shared by different users on the network and method invocations that are side-effecting are eventually replicated to all users that contain an instance, or replica, of the object. Replication is provided transparently by the runtime, so the user doesn’t need their own backing store, differing from approoaches such as the previously discussed, IPA from Holt et al.</p>
<p>Each object in Guesstimate stores two states: a “guestimated” state, which is the result of local side-effecting method invocations and a “committed” state, which is the result of atomically committed state taken from the updates each node has made to their own side-effecting “guestimated” state. These method invocations that cause side-effects must take a particular form: they must validate the change against the current state using a guard and return either true or false, depending on if the state mutation has taken effect. To verify methods take this form, the Boogie verification language is used to analyze the implementation. These method invocations must take a particular form: a method that causes modifications to the local “guestimated” state and a delegate (read: anonymous function) to be invoked when the state is finally committed on all of the nodes.</p>
<p>As nodes modify their local “guestimated” state, periodically a desginated “master” node begins synchronization rounds. These synchronization rounds begin at the master, walk each of the nodes in turn preventing updates from occuring while the synchronization round runs, and aggregates the pending updates from each nodes “guestimated“ state. These updates are stored in a tentative log that’s aggregated at the master. Once the master node aggregates all of the updates from each node, the updates, each with a pair consisting of the node identifier and operation identifier, using lexicographical ordering on the set of pairs. This determines the commit order for each update, and once established, this commit order and the associated updates are sent to each of the nodes. Once these updates are accepted by all nodes, they are then applied to the “committed” state, the delegates invoked, and the lock on updates released, so local updates are allowed to resume.</p>
<p>The ideal way of programming with Guesstimate is to have operations modify “guestimated” state first and update an associated UI afterwards. Then, the delegate is invoked and should be used to update the UI with a notification on whether or not the update has been successfully applied or was refused during committment and must be tried again.</p>
<p>One problem that can occur is issues around update interleaving at committment time. Consider the case of a ride sharing application from the paper: a guard for a method invocation that states that a user should get a ride from driver X may yieldd the user getting a ride from driver Y under a particular update reordering: therefore, guards should be written in a way where any acceptable outcome is allowed.</p>
<p>To address the problems of ordering, Guesstimate provides atomic operations: if two updates have to happen together, or there’s a causal relationship between updates (think: references, pointers, secondary indexes), these operations can be grouped together in the programming model to ensure that they commit together. To address operations that must commit before proceeding, Guesstimate provides a primitive for blocking an operation until committment.</p>
<p>Users of Guesstimate have to also deal with the reality that updates will be executed multiple times, although against different components of the objects state: “guesstimated” and ”committed”.</p>
<p>Guesstimates’ runtime takes care of most of the synchronization concerns: nodes are allowed to leave and join the system, where failed nodes are evicted after a certain amount of time and forced to rejoin the cluster and repopulate their state. Nodes form a full mesh, where each node can talk to each other node, and there’s currently no mechanism outlined for handling the failure of a “master” node that begins and coordinates the synchronization rounds.</p>
<p>We can see the genesis of ideas that made it into systems such as the Global Sequence Protocol and CAPtain, which are covered in other articles.</p>
<div id="refs" class="references">
<div id="ref-rajan2010guesstimate">
<p>Rajan, Kaushik, Sriram Rajamani, and Shashank Yaduvanshi. 2010. “Guesstimate: A Programming Model for Collaborative Distributed Systems.” In <em>ACM Sigplan Notices</em>, 45:210–20. 6. ACM.</p>
</div>
</div>
