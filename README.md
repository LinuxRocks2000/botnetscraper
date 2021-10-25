# botnetscraper
Collab project for a webscraping CPU-sharing adaptive decentralized botnet

The idea is to be able to webscrape *simulatenously* with decentralized adaptive servers (in other words, servers can join and leave without destroying the net) managing clients, which do the actual computation (running a server as a client as well is possible). In this way we can webscrape large swaths of the Internet without waiting years for our individual computers to do it, all of our computers do it together.

## Rough ideas for the control flow:
Outline of the basics of how this works in different situations.
### Client standard processes:
 * Client connects to the server
 * Client sends a "Hello" message
 * Server asks the other clients, via "peer is dry" message, for a URL to scrape
 * Server sends the client the IP addresses of its peers, so if it goes offlinet the client can reconnect
 * Server sends the client the URL to begin scraping
 * Client HTTP GETS that URL and searches for links that it can follow
 * Client, iterating through the list of links, sends the first one to the server
 * Server checks the shared URLS list. If it doesn't have that in its list, it sends it to the other servers. If it does, it tells the client not to scrape that URL, otherwise it gives the go-ahead. This automatically makes a computer-power-segregation system, as faster more powerful computers with better network connections will scrape websites first. **BUG: We need to make sure to follow a tree-scrape method, not a linear scrape, so no domain is ever abandoned.**
 * Client either abandons or scrapes the url based on the server response
### Client tree is empty (last scraped websites on the tree didn't provide any URLS):
 * Client sends an "I'm dry" message to the server
 * Server sends "peer is dry" messages to the other clients
 * First URL the Server recieves with a "water" message is sent to the original client
### Server wants to join the network:
Note that, although the said server will be using client sockets, it does not count as a client, as it doesn't do computation, it simply stores URLS and processes information from the clients.
 * Server obtains the IP of a peer, connects to it with an "I want to join" message, the peer sends it the IP addresses of the others (in case it goes offline, the new guy can reconnect to the others) and sends it the full list of scraped URLS, which is consistently synchronized between servers (see first).
 * If we want to optimize this for minimal server load (potentially weighted - we'll cross that bridge when it comes); Peer server sends a "new server" message to all the other servers, and they tell their clients to "pause" and then mathematically determine which servers will now have how many clients, then when necessary tell clients to "switch server" and new IP. Switching clients send to their new server a "switching server" message, and thus the numbers shuffle. This shouldn't happen when clients go offline or come online, only when servers come online or go offline.
### Server rejoins after disconnect:
 * Server checks through the list of peer IP addresses for one that is online, then attaches and sends an "I'm back" message (note that clients never have to do this!)
 * Peer server updates the other's URL and IP lists
 * Potentially, the servers shuffle clients
### Server disconnects:
 * Potentially, the servers shuffle clients
