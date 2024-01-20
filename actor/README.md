
Search
Write

Fadi S Kaba
Top highlight

How did PayPal handle a billion daily transactions with eight virtual machines?
Nidhey Indurkar
Nidhey Indurkar

·
Follow

7 min read
·
Jan 1
2.6K


29





I recently came across a reddit post that caught my attention: ‘How PayPal Scaled to Billions of Transactions Daily Using Just 8VMs’. Intrigued by this incredible feat, I delved into the depths of the internet, devouring PayPal engineers’ blogs and various articles. My quest was to uncover the magic behind this achievement.

Today, I’m excited to share my findings with you in this blog. But here’s the twist — we won’t just talk about it; we’ll also dive into some Go code to gain a hands-on understanding of the technology that made this remarkable milestone possible.

So, let’s start this journey together and explore how PayPal conquered the billion transactions challenge with just eight virtual machines.

December 1998 — Palo Alto, California, United States
A team of engineers (Max Levchin, Joe Lubin, and Zach Simons) created an online payment service and called it PayPal.

As they experienced a surge in popularity during its early days, attracting an ever-growing number of users, the platform encountered a substantial increase in traffic. To cope with this heightened demand and ensure seamless user experiences, they strategically invested in and bought newer hardware to scale.

However, the rapid growth didn’t slow down. In just the next two years, they hit a remarkable milestone of 1 million transactions a day. To keep up with this explosive growth, they expanded their infrastructure by running services in more than 1000 virtual machines.

While this move successfully addressed their scalability challenges, it introduced new complexities and issues that arose from managing a significantly larger and more intricate system.

Here are some of the challenges they faced in scaling to over 1 million transactions a day:

1. Network Infrastructure:
The increased demand meant that a request took more network hops to finish, leading to worsened latency. Moreover, maintaining the expanded network infrastructure became more expensive.

2. Maintenance Costs:
Adding more servers to the system increased its overall complexity. Deploying services across every machine took more time, and setting up autoscaling required additional effort. Infrastructure management tasks, such as monitoring, became more challenging.

3. Resource Usage:
Despite the expansion, they didn’t fully utilize the CPU of each server. In simpler terms, the server throughput was low, resulting in resource wastage and extra costs.

Then how did they tackle these challenges?

Adoption of Actor Model
Here’s how they managed to tackle the challenges:

Recognizing their existing code’s inefficiency in harnessing hardware capabilities, PayPal prioritized simplicity and scalability. To achieve this, they transitioned to the actor model based on the Akka framework.

This paradigm shift enabled concurrent processing, optimizing hardware use.

The actor model serves as a conceptual framework for concurrent computation, where the fundamental unit of computation is known as an actor. And here’s how the actor model offers extreme scalability:

Actors, being lightweight objects, consume fewer resources than threads, allowing the creation of millions of actors easily. Threads are dynamically assigned to actors for message processing, with each thread handling multiple actors sequentially. The number of threads scales with the CPU cores, efficiently managing a large number of concurrent actors.

Efficient Resource Utilization:
Actors, characterized by their lightweight nature, consume fewer resources than threads, enabling the effortless creation of millions of actors.

Actors with Messages Are Assigned a Thread
Threads are dynamically assigned to actors for sequential message processing, with the number of threads scaling proportionally with CPU cores, effectively managing a substantial number of concurrent actors.

Isolated State Management:
Actors maintain isolated and private states, avoiding shared memory. Communication between actors occurs through simple, immutable messages transmitted over the network.

Actors Communicating Through Messages
Each actor possesses a mailbox, functioning as a FIFO message queue for streamlined processing. Storing local state in the application server minimizes network calls to distributed caches or databases, enhancing overall system performance.

Besides PayPal uses consistent hashing to route a customer to the same server.

Effective Concurrency Handling:
The actor model facilitates simultaneous execution of multiple actors, each processing messages sequentially. This approach ensures that each actor manages only one message at a time, fostering parallelism. Asynchronous operations eliminate the need to wait for responses, simplifying concurrency management.

An Actor Process Messages in Sequential Order
PayPal leverages Akka’s functional programming style for scalability, preventing side effects and facilitating testing. The incorporation of pluggable code pieces further streamlines scalability, allowing actors to run seamlessly either locally or remotely, without requiring explicit system awareness.

Robust Fault Tolerance Mechanisms:
Actors play a crucial role in maintaining fault tolerance by creating and supervising other actors. In the event of an actor failure, the supervisor either restarts the actor or reroutes the message to an alternative actor.

Fault Tolerance in Actor Model
Error propagation to the supervisor ensures graceful error handling without unnecessary code complexity. The actor model, therefore, establishes a resilient framework for managing faults within the system.

As a result, PayPal successfully managed a billion daily transactions with remarkable efficiency, utilizing only 8 virtual machines. The move not only addressed previous challenges but showcased the power of a well-designed, concurrent model in enhancing scalability and system performance.

Now, enough of theory — let’s roll up our sleeves and dive into some Go code to unravel the Actor Model and witness its practical implementation.

Actor Model in Go
The actor model in Go Lang is a concurrency model that treats actors as independent entities, each with its own state and behaviour. GoLang provides a lightweight concurrency mechanism through goroutines and channels, which can be used to implement the actor model.

In this context, an actor is a concurrent, independent unit of computation that communicates with other actors through message passing. Goroutines can be used to represent actors, and channels facilitate communication between them. Each actor has its own state and processes messages asynchronously.

To implement the actor model in Go Lang, you would typically create a goroutine for each actor and use channels to send messages between actors. The messages would contain information about the intended action or behaviour of the receiving actor.

Here’s a simple example to illustrate the concept:

package main

import (
 "fmt"
 "sync"
)

// Actor represents an actor with its own state and a channel for receiving messages.
type Actor struct {
 state int
 mailbox chan int
}

// NewActor creates a new actor with an initial state.
func NewActor(initialState int) *Actor {
 return &Actor{
  state: initialState,
  mailbox: make(chan int),
 }
}

// ProcessMessage processes a message by updating the actor's state.
func (a *Actor) ProcessMessage(message int) {
 fmt.Printf("Actor %d processing message: %d\n", a.state, message)
 a.state += message
}

// Run simulates the actor's runtime by continuously processing messages from the mailbox.
func (a *Actor) Run(wg *sync.WaitGroup) {
 defer wg.Done()
 for {
  message := <-a.mailbox
  a.ProcessMessage(message)
 }
}

// System represents the actor system managing multiple actors.
type System struct {
 actors []*Actor
}

// NewSystem creates a new actor system with a given number of actors.
func NewSystem(numActors int) *System {
 system := &System{}
 for i := 1; i <= numActors; i++ {
  actor := NewActor(i)
  system.actors = append(system.actors, actor)
  go actor.Run(nil)
 }
 return system
}

// SendMessage sends a message to a randomly selected actor in the system.
func (s *System) SendMessage(message int) {
 actorIndex := message % len(s.actors)
 s.actors[actorIndex].mailbox <- message
}

func main() {
 // Create an actor system with 3 actors.
 actorSystem := NewSystem(3)

 // Send messages to the actors concurrently.
 var wg sync.WaitGroup
 for i := 1; i <= 5; i++ {
  wg.Add(1)
  go func(message int) {
   defer wg.Done()
   actorSystem.SendMessage(message)
  }(i)
 }

 // Wait for all messages to be processed.
 wg.Wait()
}
This program creates a simple actor system with three actors. Messages are sent concurrently to the actors, and each actor updates its state based on the received messages. The program demonstrates the actor model principles of isolation, concurrent processing, and message passing.

Below is the output generated by the provided Go code.

NOTE: Keep in mind that the specific output may vary based on the execution goroutine execution. This is a basic example, and in a real-world scenario, you might want to handle actor termination, supervision, and other aspects of the actor model. Libraries like github.com/AsynkronIT/protoactor-go provide more advanced actor model implementations in Go Lang.

Actor 3 processing message: 5
Actor 8 processing message: 2
Actor 1 processing message: 3
Actor 2 processing message: 1
Actor 3 processing message: 4
In this blog, we explored how PayPal handled a massive billion daily transactions using only eight virtual machines. We started with PayPal’s early days and the challenges they faced as their popularity soared. To tackle the increased demand, they invested in new hardware but soon hit a million transactions a day. The solution? They scaled up to over 1000 virtual machines. However, this brought new problems like network issues and high costs.

Then came the Actor Model — a cool way to manage things. In simple words, it’s like having actors (or workers) who talk to each other by passing messages. This helped PayPal become super-efficient, and we explained how it worked with lightweight actors and smart concurrency.

To make things more fun, we even wrote a simple Go code to show how the Actor Model works in action. The code creates actors, sends them messages, and lets them do their thing.

Now, with this journey complete, I hope you’ve gained a clearer picture of how systems like PayPal’s can handle big tasks. Stay tuned for more adventures into the world of tech and systems!

References:

squbs: A New, Reactive Way for PayPal to Build Applications
medium.com

Akka: build concurrent, distributed, and resilient message-driven applications for Java and Scala
Build powerful reactive, concurrent, and distributed applications in Java and Scala
akka.io

The actor model in 10 minutes
The actor model in 10 minutes
The actor model in 10 minuteswww.brianstorti.com


PayPal
System Design Interview
Actor Model
Go
Scale
2.6K


29




Nidhey Indurkar
Written by Nidhey Indurkar
338 Followers
Follow

More from Nidhey Indurkar
An Introduction to Apache Kafka: Solving Real-Time Data Streaming Challenges
Nidhey Indurkar
Nidhey Indurkar

An Introduction to Apache Kafka: Solving Real-Time Data Streaming Challenges
In the fast-paced realm of software engineering, data overload can feel like a relentless avalanche.
11 min read
·
Oct 21, 2023
175



No More Traffic Jams: Why Your Apps Need Rate Limiting
Nidhey Indurkar
Nidhey Indurkar

No More Traffic Jams: Why Your Apps Need Rate Limiting
In a world where both humans and bots are constantly tapping into digital services, an unchecked flow of requests can lead to the digital…
9 min read
·
Nov 11, 2023
38



Code Smarter: Exponential Backoff in Go Made Easy
Nidhey Indurkar
Nidhey Indurkar

Code Smarter: Exponential Backoff in Go Made Easy
Imagine The Amazon Great Indian Sale is a whirlwind of activity, with millions of users hunting for deals, creating a peak traffic scenario…
6 min read
·
Nov 3, 2023
12



Unlocking Seamless Kubernetes Deployment: A Go Backend Journey with Encore’s Automation
Nidhey Indurkar
Nidhey Indurkar

Unlocking Seamless Kubernetes Deployment: A Go Backend Journey with Encore’s Automation
Today, we’ll explore an automated approach to provision a Kubernetes cluster and deploy a Go backend using Encore. This comprehensive guide…
7 min read
·
Dec 18, 2023
15



See all from Nidhey Indurkar
Recommended from Medium
GitHub Copilot code suggestions
Jacob Bennett
Jacob Bennett

in

Level Up Coding

The 5 paid subscriptions I actually use in 2024 as a software engineer
Tools I use that are cheaper than Netflix

·
5 min read
·
Jan 5
3.3K

46



Can PostgreSQL with its JSONB column type replace MongoDB?
Yuriy Ivon
Yuriy Ivon

Can PostgreSQL with its JSONB column type replace MongoDB?
PostgreSQL is an excellent database engine, which is extremely popular these days. In my opinion, this popularity is well-deserved, as it…
18 min read
·
Aug 3, 2023
880

13



Lists



General Coding Knowledge
20 stories
·
811 saves



Visual Storytellers Playlist
55 stories
·
184 saves



Natural Language Processing
1109 stories
·
581 saves
Top 15 Software Development Trends in 2024
Serokell
Serokell

Top 15 Software Development Trends in 2024
As we step into 2024, the landscape of software development continues to evolve exponentially, driven by technological innovations and…
12 min read
·
Dec 29, 2023
1.4K

14



Why Are There So Many Programmers Who Cannot Find Jobs?
Jesús Lagares
Jesús Lagares

in

Python in Plain English

Why Are There So Many Programmers Who Cannot Find Jobs?
💸 Why code sniffing is no longer worth obtaining $100.000/y salaries.

·
4 min read
·
Dec 7, 2023
1.7K

46



Is Mamba the End of ChatGPT As We Know It?
Ignacio de Gregorio
Ignacio de Gregorio

in

Towards AI

Is Mamba the End of ChatGPT As We Know It?
The Great New Question

·
8 min read
·
Jan 12
4.6K

56



I Failed as a Lead Developer. What I’ve Learned?
Tomas Svojanovsky
Tomas Svojanovsky

in

Stackademic

I Failed as a Lead Developer. What I’ve Learned?
Navigating the Transition: Lessons Learned on the Journey from Senior Developer to Lead Developer

·
4 min read
·
Dec 26, 2023
1.4K

25



See more recommendations
Help

Status

About

Careers

Blog

Privacy

Terms

Text to speech

Teams

Reference: https://medium.com/@nidhey29/how-did-paypal-handle-a-billion-daily-transactions-with-eight-virtual-machines-76b09ce5455c
