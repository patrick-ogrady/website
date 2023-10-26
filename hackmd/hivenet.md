# Hivenet: Decentralized and Autonomous URL Recommender System Based on Useful Originality

_Originally published as a final project in December 2018 for [CS191W at Stanford University](https://explorecourses.stanford.edu/search?view=catalog&filter-coursestatus-Active=on&page=0&catalog=&q=CS191W&collapse=)_

## Abstract

An alternative to StumbleUpon, Hivenet is an implementation of a decentralized and autonomous URL recommender system that proposes a novel method to assign reputation to untrusted peers in a peer-to-peer network: useful originality. To provide context for Hivenet, this paper provides an overview of recommender systems and outlines how existing centralized recommendation systems require users to sacrifice their privacy, expose themselves to bias injection, and give up the ability to export activity data to other (competing) services. Next, this paper proposes a design paradigm on which to base decentralized recommender systems and describes the implementation of two iterations of Hivenet, which is based on this design paradigm. The latter implementation ([code](https://github.com/patrick-ogrady/hivenet)), and the main product of this paper, autonomously determines the reputation of untrusted peers by calculating the usefulness of each respective peer’s originality, assigning a unit of reputation to the first peer that discovered a URL that the user rated highly. Finally, this implementation is subjected to a simulation environment that supports the hypothesis that basing reputation on useful originality protects users from malicious peers under reasonable modeling assumptions.

## Stumbleupon: “Click a Button, Find Something Cool”

Garrett Camp, the creator of Stumbleupon, described the early internet service in a simple phrase: “Click a button, find something cool.” [1] Created in 2001, Stumbleupon solved the problem of traversing an ever growing internet by recommending content that it thought users would enjoy based on the ratings of other users that liked similar content. What originally started as an idea shared between friends at the University of Calgary eventually grew to 40 million users that “stumbled” over 60 billion times and drove half of all social traffic on the web. [2] 

In 2018, however, Garrett Camp announced that the service would shut down amid the increasing competition from social networks. At its closure, StumbleUpon drove only 0.1% of social traffic on the web (which decreased 50% YoY in its last year of existence). [3] A postmortem of the StumbleUpon company and its declining popularity, however, is not the subject of this paper. Rather, this paper proposes how a StumbleUpon clone could be built without relying on any centralized party to collect content ratings, detect malicious peers, and provide content recommendations.

### What is a Recommender System?

At its core, StumbleUpon was a recommender system. Jure Leskovec, a professor at Stanford, defines a recommender system as an application that “predicts user responses to options”. [4] For example, an online retailer might use a recommender system to predict a shopper’s willingness to purchase any of the millions of items it sells and then recommend the top 5 most attractive items on a shopper’s homepage. Likewise, a news website could predict the top articles a given visitor is willing to read based on what they have read in the past. 

### Content-Based versus Collaborative Filtering

There are two basic types of recommender systems: content-based systems and collaborative filtering systems. Content-based systems rely on the inherent properties of items that a user has previously interacted with to make predictions. For example, a user that has watched multiple western movies might be predicted to enjoy other western movies.  Pandora engineers manually determine hundreds of properties about a song to provide content-based recommendations to their listeners. [5] If a user likes a song, different algorithms can determine which properties are correlated with higher user ratings. 

Collaborative filtering, on the other hand, recommends things to a user that were preferred by similar users without regard to the inherent properties of the content itself. This latter approach has garnered significant attention in recent years as the explosive growth of content on the internet has made it increasingly difficult to extract useful information from all available online information. Given that 2.5 quintillion bytes of data are created each day, the approach of cataloging the inherent properties of chunks of data is simply not scalable to work for the majority of web data. [6] While AI techniques have been created to automatically process data, this paper explores the notion of what can be done without assigning properties to any content and instead relying solely on the behavior of users to provide useful recommendations to similar users. This paper takes this approach and explores what can be done if no attention is paid to content properties when making a recommendation system.

## Issues with Collaborative Filtering Recommender Systems

Collaborative filtering recommender systems, as noted previously, predict how a user will rate unseen items based on the preferences of similar users. To perform these predictions, a method called Singular Value Decomposition (SVD) is often used on the matrix of user data observed by the operator of the system. SVD was popularized in 2009 when a team called BellKor used SVD to win a competition to improve the recommendation accuracy of Netflix’s Cinematch system in the Netflix Prize [7]. BellKor’s method represented a 10% improvement over Netflix’s already state-of-the-art system.

![Generalized architecture of a centralized recommender system](https://hackmd.io/_uploads/BkyJurDzp.png)

_Generalized architecture of a centralized recommender system_

This matrix of user data, to no surprise, can be extremely large. Consider that Spotify has over 100 million users and 50 million songs. When represented as an $m * n$ matrix where $m = \#$ of users and $n = \#$ of times song played, this matrix is composed of 5 quadrillion cells. While most cells are sparse (indicating a user has not interacted with an item), this size of a dataset requires significant storage and computing resources to perform operations on. For this reason, companies require users to submit all their ratings to said data center to receive personalized recommendations. Such a design decision requires significant concessions from users and provides owners of user data with incredible power and protection from competitors.

### Lack of Privacy

To receive personalized recommendations, users must sacrifice their privacy and control when they provide full access to all previous activity to some recommendation generator (often a company). Privacy is sacrificed in that the company knows all of a user’s previous activity associated with a particular name or email and control is sacrificed in that said company can do whatever they choose with said activity data. In the case of Netflix, this means that a user must tell Netflix whether or not they enjoyed certain content. In the case of Amazon, this means that a user tells Amazon if they purchased a particular item. While companies that collect activity data promise to use this activity data extremely cautiously, recent news regarding Cambridge Analytica’s use of Facebook user data to run ads to flip voting preferences has certainly shown that this is not a guarantee (even when precautions are taken). [8]

### Bias Injection

Companies that operate large recommender systems have begun to consider their recommendation algorithms as trade secrets, refusing to even explain the function of algorithms to national governments. [9] Because recommendations are generated without transparency, said companies reserve the right to inject various biases into the recommendations without a user’s knowledge or consent. While most biases seem harmless, in that Amazon can recommend products that they make them more money or Spotify can recommend songs that cost them less to stream, some companies have taken this to different levels. Facebook received significant media criticism a few years ago when they published a report that “aimed to determine whether the company could alter the emotional state of its users.” [10] In this experiment, Facebook found ways to alter recommendations to make users sad or happy, without any consent or knowledge from the users involved.

### Lack of Portability

By agreeing to use most online services, a user agrees to allow the online service to collect and store a variety of data. These online services possess the full right to use such data but do not allow for such data to be easily exported. This creates a natural moat that stifles innovation and competitors from replacing stagnant products or companies. Consider that if someone wanted to start another social network, they could not just ask users to provide all the data they had given to Facebook over the past decade. These users would have to rebuild their entire online social presence on the new site. This sort of friction means that Facebook can preserve its position as a market leader simply by nature that it is so burdensome to take data out of Facebook. There is some parallel to most other online services (Spotify music history, Netflix viewing history, Google search history). The effect of this lack of portability and its monopoly-like effects has been written about extensively. [19]

## Transitioning to Decentralized Recommendations

What if users could receive transparently-produced personalized recommendations without sacrificing their privacy or portability of their data? This paper proposes a design paradigm to create such a system and provides an overview of two implementations that adhere to this paradigm.

### Proposed Paradigm

![Illustration of the proposed paradigm and services operated by each client that uses such a system](https://hackmd.io/_uploads/HJ0B9rwzp.png)

_Illustration of the proposed paradigm and services operated by each client that uses such a system_

Whereas most online services allow the user to offload all storage and computation requirements to some data center, a decentralized service requires the user to store all information that is necessary to provide any functionality and perform whatever computations that functionality demands on their local machine. The paradigm we propose relies on two major components: the interface and the agent.

#### Interface

![In the case of URLs, the interface is a Chrome browser extension](https://hackmd.io/_uploads/Hy2FqBDMp.png)

_In the case of URLs, the interface is a Chrome browser extension_

The interface in the paradigm allows a user to rate content in some viewer (ex: web browser) and communicate with a locally-operated agent over a REST API. At the minimum, this interface should have a way to redirect the user to a recommended page and rate some URL (that was visited manually or was recommended by the agent).

#### Agent

All logic required to generate recommendations, communicate with other peers, and store data occurs in the agent. This modular separation between interface and core logic allows for plug-in-play interfaces or agents depending on the desired behavior. For example, a user could prefer a strictly privacy-preserving agent over some agent that broadcasts all rating activity across a P2P network. All that is required is that any agent complies with the REST API specification (in appendix). There is no requirement, however, that all agents interoperate with each other. There is too wide an array of use cases to restrict creativity and require certain message formats or functionality beyond a core set of APIs that allow for different interfaces.

### Useful Peer-to-Peer Modules in the Context of Hivenet

#### Inter-Planetary File System (IPFS)

The InterPlanetary File System (IPFS) is a “peer-to-peer hypermedia protocol to make the web faster, safer, and more open.” [11] Instead of using URLs to address content like the web today (which isn’t related to the content at all), IPFS uses a content-addressable scheme where everything in the network has an address related to the merkle hash of said content. This abstraction allows the complexities of P2P file sharing to be abstracted to a simple “store” and “get” call across the entire IPFS network. All that is required is that each user operates an IPFS node that automatically communicates with other peers across varying network types using what supporters call “networking magic”. Fortunately, there is a pure JS implementation of this node that can be seamlessly packaged and run with any agent installation without complicated user interaction.

##### Pinning Files

Because there is no centralized store of information, the retrievability of data depends on the willingness of peers to store and serve said data (“pinning”). In the context of this paradigm, it makes sense to pin any created or retrieved messages for backup and sharing with other peers.

##### End-to-End Encryption Required

All information on IPFS is public. The only way that any data can be confidentially transmitted between peers is to encrypt everything before putting it on the IPFS network. Currently, there exists no access control functionality built into IPFS that restricts who can retrieve which content.

##### Unstable Address (so far!)

The Go Implementation of the IPFS protocol has a fully operable version of the Inter-Planetary Name System (IPNS) that allows “for creating and updating mutable links to IPFS content.” [12] This means that a user could periodically push mutated content without changing the address that content is at. For example, an IPNS address could hold the most recent rating backup of a particular user. This address would be the same even as new backups were generated and would mean that a user would not have to share a new address with all interested peers each time they updated their ratings of content. However, in the current version of JS-IPFS (as of December 2018) this feature is not yet available.

##### Global Pubsub

One especially neat feature of IPFS is that it supports publisher-subscriber (pubsub) functionality on top of the underlying file system. In essence, a pubsub system allows peers to flood the network with messages that are greedily routed to all peers that listen to a specific subscription (floodsub). This message could contain a new rating that a particular peer found or the name of a malicious peer that a certain user wants to warn other peers about. This means that peers don’t need to find some other way to communicate with each other to express that they published new ratings.

#### Chainpoint Network

Chainpoint is “an open standard for creating a timestamp proof of any data, file, or process.” [14] By adding the SHA256 hash of a piece of data to a block in a public blockchain (this process is called “anchoring”), anyone can verify said data existed at the particular time that the block was created. Because the data is added to an immutable append-only datastore (the blockchain), this ensures that the time that a hash was logged cannot be altered. In the case of this paradigm, this allows anyone to prove that they discovered a particular piece of content at a particular time.

While one could anchor data to a public blockchain without using Chainpoint, it would require paying a fee for each hash added. Given that the transaction fee on each Bitcoin transaction reached over $35 in late 2017 [15], this fee requirement would make performing anchoring cost prohibitive except for very important documents. Chainpoint allows miners to combine an unlimited number of hashes into a single merkle root that can then be added to the public blockchain. This greatly reduces the cost of submitting any single hash to the public blockchain and allows Chainpoint miners to offer the service for free. Chainpoint miners then provide a proof of data existence that allows anyone to confirm the existence of a particular hash at a particular time given the anchored merkle root.

## Building a URL Recommendation System: Private vs Autonomous

Within the constraints of the previously proposed paradigm, we detail two stages of development of a URL recommender system called Hivenet that explore tradeoffs between privacy and ease-of-use (in the sense that recommendations are automatically generated from undirected communication with other peers). These implementations will use a Google Chrome extension as the interface and provide two different agent implementations based on these specified goals. The specific logic employed by each agent is available in the appendix and a fully operable implementation of the second implementation is available on [GitHub](https://github.com/patrick-ogrady/hivenet). For both implementations it is assumed that the agent uses a collaborative-filtering based recommender system on all URLs that aren’t excluded from consideration due to being considered potentially malicious.

### Hivenet V0: Explicit Rating Sharing

![Hivenet V0](https://hackmd.io/_uploads/SkpFsrvz6.png)

The first iteration of Hivenet relies upon explicit rating sharing via access tokens over the IPFS network. This means that each user locally stores their ratings and then periodically shares the content address and decryption key of an encrypted ratings file stored on IPFS with select peers. Due to the lack of stable addressing in the JS implementation of IPFS, a new access token must be shared with peers each time ratings data is updated or the encryption key is changed.

Because users of Hivenet V0 only share rating data by explicitly providing an access token to other peers, there is no way that peers can ever retrieve verifiably-attributable ratings data from a peer that did not share an access token with them (used to deter unauthorized re-sharing of rating data). This design means that a user must find peers to utilize the service and then request their rating history over some other medium that allows for access token exchange (i.e. a messaging application). Once ratings are compiled from other peers, the agent can generate personalized recommendations.

#### Attack Detection and Mitigation on Hivenet V0

_Before reading section, it is advised that the reader fa “Hivenet V0 Implementation Guide” in the appendix._

##### Unauthorized Re-Sharing

A major risk of making content publicly available on a P2P network is that access can only be controlled via some encryption scheme. When someone is given the encryption key to something publicly accessible, there is no way to revoke access or remove the file from the network. It follows that a malicious peer could convince a user to share an access token to their rating directory and go on to share this access token with anyone who wanted it. This possibility would certainly give users pause before using such a system (if they believed their ratings and privacy had value). 

To deter this sort of attack, users do not sign encrypted ratings files with a publicly verifiable key that could allow anyone to verify the authenticity of ratings. This means that if a peer published an access token and claimed it belonged to a particular user, there would be no way to verify this claim. The only way to have any assurance of the authenticity of ratings is to request the access token directly from the user you wish to exchange ratings with (although it is true they could lie and share an access token from another user). In this sense, the only protection employed is to prevent the verification of rating ownership.

#### Weaknesses of V0 Design

##### Manual Access Token Sharing

Although strictly privacy-preserving, the manual sharing of access tokens is very cumbersome (especially because they must be shared frequently with IPNS functionality). To even use the system and generate recommendations, a new user must convince a cohort of friends to try the system and share their ratings with them. This is in contrast to existing centralized recommender systems that can offer personalized recommendations with minimal effort from new users to find ratings data by using all previously collected data. 

##### Lack of Privacy with Peers

While rating data sharing requires explicit token generation and sharing, there is in some sense less privacy than a centralized system with respect to peer observability of ratings data. For example, Spotify does not share with each user the rating history of all other user rating data that was used to generate personalized recommendations. However, Hivenet V0 requires contacting peers of interest that give full visibility into their entire rating history. In a sense, this action removes a level of anonymization that most users might require to use the system. It begs the question if a user would prefer their friends or a company know about everything they do on a service.

##### Blacklisting

![V0 interface with “blacklist” feature](https://hackmd.io/_uploads/HyP-nSwG6.png)

_V0 interface with “blacklist” feature_


The V0 system relies on the notion of user “blacklisting” to warn against malicious content. This is the case because Hivenet V0 does not employ a very sophisticated risk scoring mechanism (and being on a blacklist of the user or one of their peers is the only way to trigger a warning). However, this “blacklist” notion is confusing to the user in that there is “bad” and “blacklist”. V1 just uses the ratings of good and bad to infer malicious content, rendering the explicit “blacklisting” of content unnecessary.

##### Malicious Content Warnings

Lastly, malicious content warnings that are generated from peer-exchanged rating data are generally ineffective in V0. Unless users exchange ratings data with thousands of peers on a continual basis, it is likely that any unseen content that a user discovers will not have been seen by other peers (this is further exaggerated by the high probability that most peers that exchange ratings offline probably have similar browsing patterns). 

The V0 system gives a risk score of 1 in the case no ratings or warnings are observed for a URL out of an abundance of caution even if peers might have rated it safe if they had seen it. In most cases, the content warning serves no purpose by rating new content items as malicious. V1 deprecates this explicit warning functionality and only uses risk scores to determine what content should be considered for recommendation to the user.

### Hivenet V1: Automatic Rating Sharing using Public Keys, Anchoring, and Pubsub

![Hivenet V1](https://hackmd.io/_uploads/ry-V2BwG6.png)

The main design goal of Hivenet V1 was to automate rating sharing between peers without compromising user security. This means that any user would only need to be concerned with rating content they viewed and the agent would ensure it provided high quality recommendations of only non-malicious URLs. The problem is that automatic rating sharing is an extremely complicated problem to solve in a trustless P2P environment. Just for starters, this problem begs the following questions: How do we prevent a peer from launching a denial of service (DoS) attack? How do we prevent a sybil attack (where a single peer creates many identities that all rate malicious content positively) where a malicious peer could manipulate collaborative filtering to direct unsuspecting users to malicious content? How do we prevent a peer from attaining a good reputation and then rating malicious content highly to manipulate the risk score of a malicious URL?

#### Work-Based Message Creation

To deter DoS attacks and content scraping (discussed in more detail in the subsequent section), a computational burden is imposed upon message broadcasting. To make this mechanism effective, there must be some source of randomness only available during message creation (so peers can’t stockpile computation to perform burst message sending) and there must be a high enough level of difficulty to ensure that users must commit resources to rating activity.

The source of randomness that is only available during message creation is the mathematical proof of content generation (provided by Chainpoint) that relies on the time the proof is submitted to Chainpoint (and can’t be predicted). To determine the level of difficulty, let us first assume that a high-end CPU (tested as of December 2018) can generate $~40,000 H/s$ on a single thread. For the average user, continuously generating this many hashes per second will use considerable energy. Instead, let us bound this hashing expectation to $5,000 H/s$. To effectively deter attacks, we propose requiring on average 10 minutes of work to send a message  (rate limited by the idea that an average user will rate about 150 URLs each day). We take inspiration from the Bitcoin whitepaper and correlate difficulty with the number of leading zeros in a hash of the provided Chainpoint proof with some randomly created “nonce” [16]. We can convert our 10 minute work requirement into a requirement to generate a hash with a given number of leading 0 bits that requires searching a certain number of “nonces” on average. We must require roughly $5,000 (H/s) * 60 (seconds\ in\ minute) * 10 (minutes) = 3,000,000$ hashes to be computed on each send to meet our 10 minute work requirement. To determine the number of leading 0s to find, we take $log2(3,000,000) \approx 21$ and thus require that users must find a nonce that leads $sha256(sha256(Chainpoint Proof), nonce)$ to start with 21 0 bits. We will refer to this number of 0s to find as $PROD\_DIFFICULTY$ in later sections of this paper.

#### Reputation as a Function of Verifiable Originality

Hivenet makes the problem of “automatic rating sharing” tractable by assigning peers a fragile reputation based on their verifiable, useful originality. In other words, peers that discovered (i.e. first to rate) content that a user rates highly are trusted more than peers that have not discovered anything useful to the user. This is mathematically defined as:

$$
u = [all\ positively\ rated\ URLs\ by\ user\ that\ user\ did\ not\ discover]
$$
$$
d(u_i,p) = 1\ if\ peer\ p\ discovered\ URL\ u_i\ else\ 0
$$
$$
R(p) = \sum_{i=0}^{|u|-1} d(u_i, p)
$$

However, this trust is fragile in that a peer can be blacklisted if a user witnesses a peer acting maliciously. When a peer is blacklisted, all its previous messages are deleted from the user’s rating datastore and all future messages received from that peer are dropped. While this mechanism sounds aggressive, good peers should never get blacklisted for rating normal content (offenses that constitute blacklisting are discussed in later sections). Malicious peers could blacklist good peers but this blacklisting is a subjective operation that does not influence other peers perspective of the peer on the network (it just means the “blacklister” will not continue to broadcast received messages from the peer).

![The original discoverer of a URL based on a verifiable Chainpoint proof](https://hackmd.io/_uploads/SyFn6HwMp.png)

_The original discoverer of a URL based on a verifiable Chainpoint proof_

The intuition behind this method of assigning reputation based on originality is that it is a very difficult process to originate useful content on the internet (especially when there is a way to limit on how much content a peer can share by a computational burden). In the context of this problem, let us consider a few alternatives. Imagine that this system only relied on proof-of-work to build reputation (instead of also taking into account originality). In this sort of system, the more peers solved computational puzzles the higher their reputation would become. However, it would be straightforward for an attacker to game this system if the system was not widely used with little effort (especially if this computation requirement was bounded to make it feasible for users to generate reputation on low-CPU devices). In a stake-based system where users could put value at stake to show their honesty that could be taken if shown to be malicious, could once again be manipulated unless the stake required was extremely high (which could deter users that did not wish to make a financial investment to use the service). Lastly, a trustline approach where users give certain peers overarching trust to act responsibly offers a clear incentive to act irresponsibly or inject bias. Consider a trustline like CNN could overwhelmingly rate CNN articles positively.

##### Calculating the Risk Score for a URL

![](https://hackmd.io/_uploads/H1UkCSPGp.png)

Given an unrated URL, a user’s agent compiles a list of all the URLs that the user rated positively that the user did not discover. Next, the earliest known rating of each URL and its respective owner is pulled from the local datastore. The ratings of other content by these discoverers is then retrieved from the local datastore. For each peer, if the rating of the URL is positive, we add 1 (and 0 otherwise) to a sum denoting the number of positive markers found. If a peer discovered multiple positive URLs, they could account for multiple values added to the sum. This sum is then divided by the total number of positively rated URLs that the user did not discover and subtracted from 1. This results in a risk score where 1 is high risk and 0 is low risk. When no reputable peers have rated a new URL positively, the risk score defaults to 1. The mathematical representation of this process is listed below:

$$
n = new\ URL
$$
$$
R(p) = (reputation\ of\ peer\ p)
$$
$$
\sum R = (sum\ of\ all\ reputations)
$$
$$
g(p) = (good\ URL\ ratings\ of\ peer\ p)
$$
$$
v(n, p) = 1\ \iff n \in g(p)\ else\ 0
$$
$$
URL\ risk\ score = 1 - \dfrac{\sum_{i=0}^{|p|-1} R(p_i)v(n,p_i)}{\sum R}
$$

This method requires that the user rate some content before recommendations are able to be served, otherwise all potential URLs to rate will have a risk score of 1 and be excluded. In the view of this paper, this is a better alternative than recommending content that could potentially be malicious (this tradeoff is explored in the simulation section of the paper). Consider an alternative implementation that randomly recommended a URL that directed a user to child pornography.

#### Miscellaneous Concerns Specific to URLs

There are a few small quirks related to URLs that must be addressed before discussing potential attacks on Hivenet V1. The first of these quirks is how to handle URL parameters. These parameters can range from access tokens for secure content to referral tracking for advertisements. A naive solution would be to simply strip all URL params, however, this causes some websites to function incorrectly. Consider that ESPN uses a game_id parameter to direct viewers to the appropriate page detailing information about that game. If URL params are not handled properly, access could be granted to restricted content or two URLs that route to identical content will be seen by the agent as too distinct URLs. With a content-addressable scheme, this is not a problem, however, URLs often do not relate to the unique hash of the content. For the sake of Hivenet, we implement a best-effort approach where common authentication and tracking parameters are stripped from the URL but fear doing anything else could cause unintended consequences on some URLs that limits proper functionality.

The second quirk that should be addressed is the importance of hostname. One could imagine using the hostname to improve the generated risk score for an unseen URL, however, most content generated online is user-generated. Hostnames are no longer indicative of a consistent viewing experience across a site. Consider the case that the ipfs.io hostname is evaluated by the system. IPFS hosts a public gateway at this address for any IPFS content. Surely this should not guarantee all items fetched from this gateway as reputable.


#### Attack Detection and Mitigation on Hivenet V1

_Before reading section, it is advised that the reader first peruses “Hivenet V1 Implementation Guide” in the appendix._

To understand the usefulness of reputation as a function of originality, we consider a variety of attack vectors on Hivenet and predict their outcome. This list is certainly not exhaustive but does aim to address significant attacks. We also introduce a series of parameters that we will optimize in the next section _“Parameter Optimization via Simulation”_.

##### Defining Parameters

* $MAX\_RISK\_TOLERANCE$ = max riskiness score of a URL to be considered safe for recommendation
* $RANDOM\_TRY\_URL$ = probability of trying a suspicious URL when no safe recommendations available
* $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS$ = amount of reputation that must be assigned to peers before consider any peer recommendations
* $PROD\_DIFFICULTY$ = number of leading zeros in sha256(sha256(Chainpoint Proof), nonce)

##### DoS Attack

A DoS attack in the context of Hivenet V1 could resemble one of two possibilities: a peer could rebroadcast the same message continuously or a peer could broadcast a new message without valid work.

In the first situation, an honest user would simply refuse to broadcast a message it had already observed. If most users in the network are honest, this protection should stimy most attacks of this type. It is important to note that IPFS pubsub offers little native protection from this sort of attack (in that an IPFS peer can be banned but a new peerID can easily be regenerated and the user can be retargeted for attack). As it pertains to blacklisting the peer associated with the broadcasted message, it would not be a good idea to blacklist such a peer because another user could rebroadcast a message like this in an attempt to get such a peer blacklisted. 

![](https://hackmd.io/_uploads/SymFyIPG6.png)

In the second situation, a peer could either be attempting to overload the network or ingest a large amount of information (see scrape attack). In any case, an invalid amount of work with a valid signature constitutes blacklisting of said peer. Future messages would be dropped.

##### Malicious Content Attack

A malicious content attack occurs when a peer positively rates malicious content from the time their account was created or after they discovered URLs rated positively by other peers. In the first case, malicious URLs that had not yet been seen would be given a high risk score and would not be recommended to any users (unless a large portion of reputation was allocated to malicious hosts). Malicious URLs that had been seen by honest peers would be rated bad and would be given a high risk score by peers that gave the rating peers reputation for their earlier discoveries. 

In the second case, a peer that performed such an attack would require the cooperation of multiple other peers that discovered original and useful content for a given user to simultaneously rate such content positive to reduce the risk score to below the $MAX\_RISK\_TOLERANCE$ tolerance the recommender requires to even consider said URL. While this attack sounds straightforward, there are two important ideas to consider. Since the reputation assigned to each peer is dependent on the user’s preferences, a peer attempting this attack could potentially require a distinct set of cooperating peers to influence each user with vastly different preferences. A user that has rated just 100 URLs positive that were discovered by other peers would have had to rate at least $100 * MAX\_RISK\_TOLERANCE$ URLs that were discovered by these maliciously cooperating peers for the URL to be considered for recommendation. Given that collaborative filtering also takes preferences of the user into account when generating the next recommendation, cooperating peers would also have to be the first to discover content (not just rate similarly) that this user would find interesting to even be considered for rating by the user.

##### Sybil Attack

In a sybil attack, an attacker  “subverts the reputation system of a peer-to-peer network by creating a large number of pseudonymous identities, using them to gain a disproportionately large influence.” [17] This large influence could be used to manipulate a URLs projected risk score or make a URL appear like a more attractive recommendation to drive malicious content or bias the recommender system. Because reputation is dependent on the usefulness of discovered URLs, creating many identities won’t increase an ability to manipulate the risk score of given URLs unless each identity discovers useful URLs. In this case, however, no more reputation will be earned than just discovering said URLs with a single identity.

In regard to manipulating collaborative filtering, however, a malicious peer could create multiple identities that mirrored the ratings of URLs broadcasted by a given user that the peer wished to manipulate with some additional URLs to bias the user towards. In this case, the target URLs that the user had not yet rated would be biased to be recommended sooner, with the opposite occurring if rated badly by the attacking peer. However, this attack would be difficult to pull off because a peer must discover at least one useful URL to a user before their ratings are considered in recommendation generation and the URL that this malicious peer(s) recommend must be declared “unrisky” by other reputable peers to even be recommended. This would take considerable time (must wait for user to rate discovered content by any peer), effort (must find interesting content that no one else has discovered), and computational resources (for each message sent must calculate nonce). If this stokes fears of an automated approach to undertaking such an attack, one should proceed to the section discussing scrape attacks.

##### Forgery Attack

In the case that a malicious peer wanted to destroy the reputation of an honest peer, they might consider forging a rating from said user. However, this is impossible to perform because each broadcasted rating is signed using the private key of a peer. Without the private key, a malicious peer could not generate a valid signature to impersonate another peer. Note, an invalid signature does not result in peer blacklisting for this reason. A malicious peer could generate an invalid signature for an honest peer, which is not a sign of maliciousness by the honest peer.

##### Scrape Attack

In an attempt to build a meaningful reputation, a peer might consider scraping all the yet to be discovered URLs on a website with the hope that a user might find one such URL useful. Because each message requires $\approx 10\ minutes$ of work, it would take a peer about 17 hours of work to scrape 100 URLs. A malicious peer could not scrape these 100 URLs faster by using multiple peer identities (the same amount of work would still be required). Furthermore, simultaneous scraping would divide reputation across multiple public keys. At this point, this would be considered a sybil attack.

##### Dynamic URL Forwarding

In the situation that a URL forwarder is rated, the controller of the forwarding could alter the destination address after rating. Because the rating is based on URLs, there is no real protection against this type of attack and agents should not allow ratings of common URL forwarders. To cause any harm, the original forwarding address must also point to something found useful (which is certainly not straightforward).

##### Stolen History Attack

![](https://hackmd.io/_uploads/Bk4zl8vMa.png)

A malicious peer might wish to steal the message history of a popular peer by publishing a message that links to said peers previous message instead of one of the malicious peers. When doing a historical crawl, it is necessary to check that fetched messages have the same public key as the signed message that started the historical crawl. In this case, the malicious public key would be blacklisted.

##### Mooch “Attack”

Based on the current formulation of Hivenet V1, a user could choose to listen to broadcasted ratings and not broadcast any of their own ratings. They could rate content but only store it locally to use to generate personalized recommendations. Because broadcasted messages are public, there is nothing stopping this sort of “attack” however it is unlikely this could comprise a large portion of the network because the less users that rate content make recommendation generation much less useful. Thus, when this argument is taken to its extreme, the entire network would fall apart. If there is any value in Hivenet to users, they would be compelled to broadcast ratings.

##### Deanonymization Attack

At no point does Hivenet V1 require the user to provide any personally identifying information (i.e. name, city, email, Facebook). In the case that a malicious peer wished to attach a real world identity to broadcasted ratings, they would somehow need to correlate a randomly-generated public key with other identifying information. Because broadcasted URLs are stripped of any user-specific information and users only broadcast URLs they wish to rate (instead of their entire browsing history for example), this feat would be difficult to accomplish from looking only at broadcasted messages with no context.

#### Parameter Optimization via Simulation

To test the efficacy of the proposed Hivenet algorithms under different scenarios, we created a simulation environment that models the behavior of agents (both honest and malicious) exchanging URL ratings. The project GitHub page provides the testing code for these simulations and allows the reader to play around with different parameters to tune the agent behavior as desired. This section examines a few example simulation scenarios and discusses conclusions taken away from them.

##### Simulation Parameters
* $PROBABILITY\_CREATE\_GOOD$ = likelihood of adding a good peer on any given round of the simulation
* $AGENT\_SIMILARITY$ = similarity of sites browsed by good peers (there must be some overlap of site visits to build reputation before trusting recommendations from unknown peers)
* $PROBABILITY\_CREATE\_BAD$ = likelihood of adding a malicious peer on any given round of the simulation
* $MALICIOUS\_AGENT\_PREDICT\_USEFUL$ = probability that a malicious agent predicts a URL that good peers will find useful in the future
* $MALICIOUS\_AGENT\_TIME\_SPENT\_GUESSING\_USEFUL$ = percentage of rounds that a malicious peer attempts to predict a useful URL

##### Modeling URL Browsing

In this simulation, we make strong assumptions about the browsing behavior of peers across the internet to create a tractable projection of algorithm performance over a series of interactive rounds. This section describes the parameters that must be configured to enforce such assumptions.

We assume the probability a good peer is created is $PROBABILITY\_CREATE\_GOOD$ and likewise $PROBABILITY\_CREATE\_BAD$ for a malicious peer. When these parameters are set equal to each other, this means that the probability that a good agent is created in any particular round of the simulation is just as likely as creating a malicious one. More abstractly this indicates that the amount of good hashpower in the network is considered equivalent to the amount of malicious hashpower in the network. Regardless, the group with the majority of hashpower is considered “good”. The probability that any malicious agent sends a blacklistable message is $PROBABILITY\_SEND\_BLACKLISTABLE\_MESSAGE$.

We assume peers take a random walk across URLs on the internet where good agents browse similar URLs to each other with a probability of $AGENT\_SIMILARITY$. We assume this number must be set above zero otherwise there would be no way for any user to start viewing recommendations of untrusted peers (recall at least $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS$ URLs created by other peers must be rated good before untrusted recommendations can be considered and all recommendations considered must have a risk score less than $MAX\_RISK\_TOLERANCE$). When good agents are not browsing similar content, we assume they are browsing content useful to them but not useful to other peers. The agent also randomly samples observed URLs with a probability of $RANDOM\_TRY\_URL$ (including potentially suspicious URLs), if no recommendation is available.

We assume that malicious peers can predict whether or not a URL will be useful to good peers with a probability of $MALICIOUS\_AGENT\_PREDICT\_USEFUL$ and assume that a malicious peer spends $MALICIOUS\_AGENT\_TIME\_SPENT\_GUESSING\_USEFUL$ of their rounds guessing potentially useful URLs. When a malicious agent is not attempting to discover some useful content, we assume they are sharing suspicious URLs.

##### Simulation Goals

In the simulation, a good peer wishes to avoid viewing suspicious URLs. A malicious peer wishes to manipulate good peers into viewing suspicious URLs or find a way to corrupt a good peer’s local datastore with invalid messages.

##### Simulation GUI Legend

* $BLUE\ NODE$ = honest peer
* $YELLOW\ NODE$ = malicious peer that has not sent an invalid message (only suspicious URLs)
* $GREEN\ NODE$ = honest peer that has viewed a suspicious URL
* $RED\ NODE$ = malicious peer that has sent an invalid message
* $BLUE/GREEN\ EDGE$ = non-red edges indicate the reputation a peer puts on another peer (the thicker the edge the more reputation)
* $RED\ EDGE$ = peer identifies another peer as malicious

##### Simulation 1: Malicious Peers Send Invalid Messages Frequently

###### Parameter Settings

* $PROBABILITY\_CREATE\_GOOD = 0.75$
* $PROBABILITY\_CREATE\_BAD = 0.75$
* $MAX\_RISK\_TOLERANCE = 0.5$
* $RANDOM\_TRY\_URL = 0.0$
* $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS = 5$
* $PROBABILITY\_SEND\_BLACKLISTABLE\_MESSAGE = 0.1$
* $AGENT\_SIMILARITY = 0.3$
* $MALICIOUS\_AGENT\_PREDICT\_USEFUL = 0.1$
* $MALICIOUS\_AGENT\_TIME\_SPENT\_GUESSING\_USEFUL = 0.8$

###### Intuition

The first simulation explores a scenario where the probability that a malicious peer sends a “blacklistable” message is set to 10% ($PROBABILITY\_SEND\_BLACKLISTABLE\_MESSAGE = 0.1$). This simulates a reality where malicious peers try to wreak havoc on Hivenet by producing a high proportion of invalid messages to steal reputation, corrupt peer databases, and share messages with insufficient work.

###### Visualization

![](https://hackmd.io/_uploads/rkXtgOPzT.png)

###### Results

Because peers have very fragile reputations (as soon as any malicious behavior is detected the peer is blacklisted), this sort of attack is very ineffective. As soon as a good peer witnesses malicious behavior, the peer deletes all received messages from the malicious peer and stops rebroadcasting their messages. Furthermore, the visualization indicates that no good peers were exposed to any suspicious URLs.

##### Simulation 2: Random Viewing of Unrecommended URLs

###### Parameter Settings

* $PROBABILITY\_CREATE\_GOOD = 0.75$
* $PROBABILITY\_CREATE\_BAD = 0.75$
* $MAX\_RISK\_TOLERANCE = 0.5$
* $RANDOM\_TRY\_URL = 0.25$
* $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS = 5$
* $PROBABILITY\_SEND\_BLACKLISTABLE\_MESSAGE = 0.01$
* $AGENT\_SIMILARITY = 0.3$
* $MALICIOUS\_AGENT\_PREDICT\_USEFUL = 0.1$
* $MALICIOUS\_AGENT\_TIME\_SPENT\_GUESSING\_USEFUL = 0.8$

###### Intuition

In this simulation, malicious peers send “blacklistable” messages with a much lower 1% probability. However, honest peers randomly attempt to view observed but unrecommended URLs when they don’t have any available recommendations with a 25% probability. This method is inspired by the notion of optimistic unchoking in BitTorrent [18] in that a peer might find a very reputable peer by randomly trying risky content. 

###### Visualization

![](https://hackmd.io/_uploads/H1I0g_PGT.png)

###### Results

This approach works well in BitTorrent because the integrity of received file chunks can be checked and dropped if invalid (with little risk of any kind to the honest user), however, with no straightforward way to check the maliciousness of delivered content (no such thing as a malicious integrity check) almost all good peers are exposed to malicious content from malicious peers. This aggressive approach would increase reputation quickly when $AGENT\_SIMILARITY$ is low, however, it greatly increases the probability of exposure to risky content.

##### Simulation 3: Low Minimum Reputation Assigned

###### Parameter Settings

* $PROBABILITY\_CREATE\_GOOD = 0.75$
* $PROBABILITY\_CREATE\_BAD = 0.75$
* $MAX\_RISK\_TOLERANCE = 0.5$
* $RANDOM\_TRY\_URL = 0$
* $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS = 0$
* $PROBABILITY\_SEND\_BLACKLISTABLE\_MESSAGE = 0.01$
* $AGENT\_SIMILARITY = 0.3$
* $MALICIOUS\_AGENT\_PREDICT\_USEFUL = 0.1$
* $MALICIOUS\_AGENT\_TIME\_SPENT\_GUESSING\_USEFUL = 0.8$

###### Intuition

In this simulation, $RANDOM\_TRY\_URL = 0.0$ and $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS = 0$. The lower $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS$, the less peer URLs must be rated good before recommendations from peers are considered. Naturally, the less peer URLs rated, the higher probability that a good peer could rate assign a malicious peer higher reputation.

###### Visualization

![](https://hackmd.io/_uploads/S1Oub_wMa.png)

###### Results

The visualization indicates that there were far less good peers exposed to suspicious content (green nodes) and most malicious peers that have not broadcasted a “blacklistable” message (yellow nodes) have low reputation. However, some good peers happened to rate content from malicious peers good before rating any good peer URLs and were exposed to suspicious URLs (when this occurs suspicious URLs receive a low risk score).

##### Simulation 4: High Minimum Reputation Assigned

###### Parameter Settings

* $PROBABILITY\_CREATE\_GOOD = 0.75$
* $PROBABILITY\_CREATE\_BAD = 0.75$
* $MAX\_RISK\_TOLERANCE = 0.5$
* $RANDOM\_TRY\_URL = 0$
* $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS = 5$
* $PROBABILITY\_SEND\_BLACKLISTABLE\_MESSAGE = 0.01$
* $AGENT\_SIMILARITY = 0.3$
* $MALICIOUS\_AGENT\_PREDICT\_USEFUL = 0.1$
* $MALICIOUS\_AGENT\_TIME\_SPENT\_GUESSING\_USEFUL = 0.8$

###### Intuition

To maintain the safety of good peers, we set $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS = 5$ in this simulation. While this requirement means that a good peer must rate more content before receiving any recommendations from untrusted peers, it is assumed this reality is favored to one where good peers can be exposed to malicious URLs (which would deem the recommender system useless).

###### Visualization

![](https://hackmd.io/_uploads/rk6AbuvGp.png)

###### Results

This simulation indicates the higher value of $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS$ the lower the probability that a good peer will view a malicious URL (when all other parameters are kept equal). Some reputation is still assigned to malicious peers but no suspicious URLs are viewed because the majority of any peer’s trust is allocated towards good peers (which in general discover more useful content than malicious peers).

### Future Work

#### Efficiently Fetching Historical Content

Historical messages are fetched by observing the last IPFS address of a given message and fetching it via IPFS. By storing these small file pieces, it should increase availability across the IPFS network (better to miss a message than an entire repository of messages), however, when a user is fetching many ratings for the first time they will have to issue potentially thousands of file fetch requests per peer to acquire their entire history.

#### Decrease Network Activity

Because recommendations are generated on each edge device, each message must be communicated to every node. This incurs INCREDIBLE network overhead compared to a centralized architecture. If there are a million peers a single message would need to be repeated across the network at least a million times to be delivered to each peer (ex: 200 byte message uses 200 MB of bandwidth across entire network assuming there is no overhead or rebroadcasting). The only way to mitigate this would be to require peers to only fetch the ratings that would be interesting to them based on peers they have assigned reputation to and then randomly sampling other peers for useful content (akin to how BitTorrent randomly unchokes peers). [18]  

#### IPFS-Pubsub Pull Request for Client Message Broadcasting Control

Currently the version of JS-IPFS that the implementation is based on (0.32.3) does not allow the client to control when messages are broadcasted to other peers and uses a pubsub flood approach [13]. Without broadcast discretion, it is impossible to provide protection from malicious participants. We propose a fork of this codebase that does not re-broadcast messages by default and instead requires the client to manually re-broadcast any message it receives and finds useful.

#### Streaming Recommender System

Collaborative filtering requires performing Singular Value Decomposition across a matrix of all observed ratings. When millions of ratings are observed, by a user across many peers, this can be both a computational burden and storage burden for the agent. There has been research into streaming recommender systems that update their recommendations with each subsequent piece of data and then discard the message. This sort of system could greatly reduce the computational and storage requirements of each agent. If we need to store each message of size 1 KB, then we can store at most 1.5 million total ratings across all observed activity (if allocated 1.5GB of storage)

#### Reward Mechanism for Originality

The notion of originality and reputation is only used to generate a risk score for unfamiliar URLs. It would be interesting to use this originality to generate a score to compete with other peers, provide improved performance on the network, or allow a given peer to bypass the work restriction on each message.

## Conclusion

This paper proposed a decentralized and autonomous alternative to StumbleUpon that uses the concept of useful originality to assign reputation to untrusted peers in a peer-to-peer environment. To illustrate the need for such a system, this paper discussed the shortcomings of centralized recommender systems: lack of privacy, bias injection, and lack of portability. Next, this paper outlined a design paradigm to use for the construction of decentralized recommender systems and detailed two iterations of design based on this paradigm. The result of the second iteration, Hivenet, autonomously determines the reputation of untrusted peers by calculating the usefulness of each respective peer’s originality, assigning a unit of reputation to the first peer that discovered a URL that the user rated highly. The performance of Hivenet was then tested in a simulation environment and showed how under reasonable assumptions that a robust reputation system could be based on originality. 

## Acknowledgements

David Mazières served as my research advisor for this paper. He offered incredible feedback and guidance throughout the project.

## Appendix

### Works Cited

[1] Waters, D. (2007, March 29). Web 2.0 wonders: StumbleUpon. Retrieved December 9, 2018, from http://news.bbc.co.uk/2/hi/technology/6506055.stm

[2] Farokhmanesh, M. (2018, May 24). Goodbye, StumbleUpon, one of the last great ways to find good things online. Retrieved December 9, 2018, from https://www.theverge.com/2018/5/24/17389230/stumbleupon-shut-down-internet-discovery

[3] Zevin, C. (2018, February 22). Pinterest, Google, & Instagram big winners as Facebook share of visits falls 8% in 2017 [REPORT]. Retrieved December 9, 2018, from https://www.shareaholic.com/blog/search-engine-social-media-traffic-trends-report-2017/

[4] Leskovec, J., Rajaraman, A., & Ullman, J. D. (2014). Mining of massive datasets. Cambridge: Cambridge University Press. Page 307.

[5] Walker, R. (2009, October 14). The Song Decoders at Pandora. Retrieved December 9, 2018, from https://www.nytimes.com/2009/10/18/magazine/18Pandora-t.html

[6] Marr, B. (2018, July 09). How Much Data Do We Create Every Day? The Mind-Blowing Stats Everyone Should Read. Retrieved December 9, 2018, from https://www.forbes.com/sites/bernardmarr/2018/05/21/how-much-data-do-we-create-every-day-the-mind-blowing-stats-everyone-should-read/#ff963dd60ba9

[7] Congratulations to team "BellKor's Pragmatic Chaos" for being awarded the $1M Grand Prize on September 21, 2009. (2009, September 18). Retrieved December 9, 2018, from https://www.netflixprize.com/community/topic_1537.html

[8] Granville, K. (2018, March 19). Facebook and Cambridge Analytica: What You Need to Know as Fallout Widens. Retrieved December 9, 2018, from https://www.nytimes.com/2018/03/19/technology/facebook-cambridge-analytica-explained.html

[9] DeMers, J. (2018, February 07). How Much Do We Really Know About Google’s Ranking Algorithm? Retrieved December 9, 2018, from https://www.forbes.com/sites/jaysondemers/2018/02/07/how-much-do-we-really-know-about-googles-ranking-algorithm/#57aab19f55bb

[10] Rushe, D. (2014, October 02). Facebook sorry – almost – for secret psychological experiment on users. Retrieved December 9, 2018, from https://www.theguardian.com/technology/2014/oct/02/facebook-sorry-secret-psychological-experiment-users

[11] IPFS Homepage. (n.d.). Retrieved December 9, 2018, from https://ipfs.io/

[12] IPNS Documentation. (n.d.). Retrieved December 9, 2018, from https://docs.ipfs.io/guides/concepts/ipns/

[13] Libp2p/js-libp2p-floodsub. (2018, December 06). Retrieved December 9, 2018, from https://github.com/libp2p/js-libp2p-floodsub

[14] Chainpoint Homepage. (n.d.). Retrieved December 9, 2018, from https://chainpoint.org/

[15] Bitcoin Transaction Fees. (n.d.). Retrieved December 9, 2018, from https://bitcoinfees.info/

[16] Nakamoto, S. (n.d.). Bitcoin: A Peer-to-Peer Electronic Cash System. Retrieved December 9, 2018, from https://bitcoin.org/bitcoin.pdf

[17] Trifa, Z. (2014, June 05). Sybil Nodes as a Mitigation Strategy Against Sybil Attack. Retrieved December 9, 2018, from https://www.sciencedirect.com/science/article/pii/S1877050914007443?via=ihub

[18] Cohen, B. (2008, January 10). The BitTorrent Protocol Specification. Retrieved December 9, 2018, from http://www.bittorrent.org/beps/bep_0003.html

[19] Cerf, M., Matz, S., & Rolnik, G. (2018, March 8). Commentary: There's Still Time to Stop the Tech Monopoly Takeover. Retrieved December 9, 2018, from http://fortune.com/2018/03/08/privacy-data-collection-facebook-google-amazon-apple-microsoft/

### Agent API Reference

We define a required interface for any agent with the idea that users can choose to swap out different interfaces and agents depending on their preferences.

#### Rating 

**POST** _/rating body:{contentID, rating}_
* User provides a rating for URL content. (kept as contentID so could be generalized to other categories
* It is implied that the user is comfortable with any rating being broadcasted/shared.

#### Get Recommendation

**GET** _/recommendations body:[{contentID, score}]_
* Request that the local agent produces a recommendation for review

#### Backup

_API Interface Optional (depends on implementation)_
* Backs up all user data (in an encrypted format to IPFS)

#### Share Ratings

_API Interface Optional (depends on implementation, could occur autonomously)_
* Must exist some functionality to exchange ratings with peers (whether manually or automatically) 

### Hivenet V0 Implementation Guide

#### Subroutines

##### Remove Unnecessary URL Params

```
Name: cleanURL
Params: OriginalURL
Returns: ModifiedURL
```

* Remove following params (comparing in lowercase):
  * sessionid, session_id
  * anything with “utm”, source
  * credential, accesstoken, access, token 
* Sort params in alphabetical order (ensures same URL for same address and params in recommendation engine)

##### Store File on IPFS

```
Name: storeString
Params: StringToStore
Returns: AccessToken (<IPFS Address>&<Randomly Generated Secret>)
```

* Encrypt serialized datastore of ratings and blacklist using a randomly generated 64-bit key using aes-256-ctr.
* Add encrypted file to IPFS.
* Return access token to user made of the form `<IPFS Address>&<Randomly Generated Secret>` for sharing.

##### Get File Stored on IPFS

```
Name: getString
Params: AccessToken
Returns: StoredString
```

* Split access token provided by storeString  on “&”
* Query IPFS network for address
* Pin IPFS file with address once retrieved (for other peers to access)
* Decrypt file using access token secret
* Return file as a string

##### Generate Recommendations

```
Name: generateRecommendations
Params: None
Returns: None
```

* Remove all URLs on anyone’s (user or other peers) blacklist.
* Perform SVD on remaining ratings
* Cache all unseen recommendations for future recommendation

#### Before Loading Site

_Calculating a risk score for unseen websites is deprecated after V0. Risk score will only be used to determine if a URL should be recommended._
* Run `cleanURL`
* Check to see if cleaned URL already rated by user
  * If yes, load page with no warnings and exit execution
* Check to see if cleaned URL is on user or any peer blacklist
  * If yes, show warning to user that cleaned URL is on a blacklist before loading page contents (if on ANY peer blacklist) with name of peer(s)
    * Continue to load page if user dismisses warning
* Check to see if cleaned URL is not on any good rating lists
  * If not on any, show warning
    * Continue to load page if user dismisses warning
  * If yes, load page contents

#### Rating

* Run `cleanURL`
* Add modified URL to local datastore of ratings
* Remove any blacklist records of modified URL
* Run `generateRecommendations`

#### Blacklist

_Explicitly blacklisting content is deprecated after V0. Risk score will be calculated using reputation generated from discovering useful content instead of via explicit URL marking._
* Run `cleanURL`
* Add modified URL to local datastore of blacklisted URLs
* Remove any rating of modified URL 

#### Get Recommendation
* If there exist computed recommendations, get cached recommendation with highest score. Remove from cached recommendations.
  * Direct browser to recommended URL.
  * Display projected rating of recommended site (scaled 0 to 1).
* Else, show warning that no recommendations available.

#### Share Ratings and Blacklist
* Run `storeString`
* Show popup with access token to share with other peers.

```
sharedObject = [[url1, rating], …, [urln, rating]] where urli is cleaned and rating == -1 if blacklisted
```

#### Import Ratings and Blacklist
* Run `getString`
* Ask user for a username to assign to records (no publicKey provided from access token)
* Remove any stored records under the provided username
* Store all ratings in local DB under a user provided username

#### Backup Data
* Run `storeString`
* Show popup with access token to use to restore data

```
backupObject = {
    imports:[[userName1, accessToken], ..., [userNamen, accessToken]],
    me:[[url1, rating], …, [urln, rating]]
}
```

#### Import Backup Data
* Run `getString`
* Delete anything in local datastore
* Import serialized records to local datastore

### Hivenet V1 Implementation Guide
_Link to GitHub: https://github.com/patrick-ogrady/hivenet_ 

#### Subroutines (including those from dRecs)

##### Generate Risk Score

```
Name: getRiskScore
Params: URL
Returns: Float from 0.0 to 1.0 (where 1 is risky)
```
* Implementation of mathematical expressions provided in paper.

##### Add to Message Queue

```
Name: processMessageQueue 
Params: URL, rating
Returns: None
```
* Run processMessage when nothing being processed

_Message queue is required because there must be a strict ordering of processed messages for each user (lastMessageIPFS must lead to a single ordering of message or will be blacklisted)._

##### Create Message

```
Name: createMessage
Params: URL, rating, lastMessageIPFS, publicKey, privateKey
Returns: formatted message
```
* Generate message given inputs
* Sign message with private key
* generate Chainpoint proof
* Find a nonce for `sha256(sha256(proof), nonce)` with $PROD\_DIFFICULTY$ 0’s
* Pin current message to IPFS (put nonce in saved message so that it can be verified for historical pull)


_The message object requires two signatures: one to attach a publicKey to the content saved in Chainpoint and one to ensure the message hasn’t been tampered with._

```
messageObject={
	signature:xxx,
    publicKey:xxx,
	messageToSend:{
        nonce:xxx,
        proof:xxx,
        message:{
            signature:xxx,
            publicKey:xxx,
            payload:{
                url:xxx,
                rating:0/1,
                lastMessageIPFS:xxx
            }
        }
	}
}
```

##### Parse Message

```
Name: parseMessage
Params: Message
Returns: None if valid, else publicKey of peer to blacklist.
```
* Check if IPFS address of content already seen
  * If so, skip
* Check to make sure signature valid
* Check to make sure Chainpoint proof valid
  * Make sure that points to correct hash
* Check to see if multiple messages for publicKey with same lastMessageIPFS
  * If so, blacklist
* Check to make sure user has generated sufficient work (given message nonce)
* Check to see if user re-rated content
  * If so, blacklist
* If lastMessageIPFS not seen, call `pullHistory` on lastMessageIPFS
  * Blacklist if `pullHistory` leads to invalid 

##### Pull Previous Messages

```
Name: pullHistory
Params: originalPublicKey, nextMessageIPFS
Returns: None if valid, else public key of peer to blacklist
```
* Check to see if already stored locally in ratings DB. 
* If not, run `getString` on provided IPFS address
* Parse received message
  * Add to DB if valid
  * Return publicKey if not valid
* Continue until already have `nextMessageIPFS` in DB

##### Process Message

```
Name: processMessage
Params: rating, url
Returns: None
```
* Create message to send over pubsub
* Add to ratings directory
* `broadcastMessage` across IPFS pubsub (if not historical) 
* Create new backup of local DB

##### Broadcast Message

```
Name: broadcastMessage
Params: message
Returns: None
```
* Send over IPFS Pubsub 

##### Generate Recommendations (overwrites same function from V0)

```
Name: generateRecommendations
Params: None
Returns: None
```
* Generate Reputations for all peers given implementation of mathematical expressions provided in paper.
* Take a subset of all rating information where every URL has either already been rated by the user or `getRiskScore` of the URL $MAX\_RISK\_TOLERANCE$ and is rated by a peer that has discovered at least one useful URL.
* Perform SVD on remaining ratings
* Delete all cached recommendations
* Cache all unseen recommendations for future recommendation
* If no recommendations available or less than $MINIMUM\_REPUTATION\_ASSIGNED\_BEFORE\_RECOMMENDATIONS$, return a random WIKI page

##### Rating
* Run `cleanURL`
* Run processMessageQueue with cleaned URL and rating of 1 for good or 0 for bad

##### Get Recommendations
* Run `generateRecommendations`
* get cached recommendations sorted by highest score 
  * Separate by those with low and high risk score
  * Once rated, remove from recommendations list

##### Receive Message over IPFS Pubsub
* Call `processMessage` on received message (will determine if should be broadcasted to other peers).

##### Backup Data
* Encrypt serialized datastore of ratings using private key
* Run `storeString`
* Store IPFS address in persistent memory

##### Import Backup Data
* Run `getString` of stored IPFS address 
  * If none exists, initialize new store
* Delete anything in local store
* Import contents into DB
