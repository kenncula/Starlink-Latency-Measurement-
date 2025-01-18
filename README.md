# Measuring Starlink Latency Using RIPE Atlas

The goal of this project is to analyze the performance of RIPE Atlas probes connected to
Starlink user terminals across the world. In this analysis, we also set out to identify possible
reasons why certain trends do (or do not) appear

## Methodology
### 1.1 Probe Identification and Selection
We first explored how to get the current, active probes through the REST endpoints that
RIPE Atlas exposes. We then found that the Starlink probes shared the ASN 14593.
From the list of current, active probes under the Starlink ASN (54 probes as of 05/01/23),
we picked 14 probes to monitor, each identified by a unique probe ID. To get a better
understanding of how the probes performed worldwide, we selected a sample of probes from
across the globe:
   - North America (4): 61537 (USA East Coast), 60929 (USA West Coast), 61113 (USA Alaska), 60510 (Canada) 
   - Australia (3): 52955 (New Zealand), 52918 (Australia West Coast), 1004453 (Australia East Coast) 
   - Caribbean (1): 1005627 (Haiti) 
   - South America (1): 26834 (Chile) 
   - Europe (5): 1004876 (Italy), 1002750 (Germany), 35681 (Austria), 17979 (UK), 20544 (Poland)
    
We then pinged bing.com from the selected probed in 15-minute intervals. Each of these
pings sent 3 packets from the probe to bing.com and allowed us to receive latency statistics.
The most valuable statistic that we received was the round-trip time (RTT) of each packet. This
pinging and data collection was performed over the course of 24 hours twice, once during the
weekend and once during the weekend. The data collection is done by RIPE Atlas and is given
some measurement ID that can be used to request a JSON from their REST API.

### 1.2 Data parsing and plotting
  With the JSONs provided in the HTTP request of the measurement results, we
programmed a script to parse through and extract the recorded RTT data (with pre-calculated
averages, mins, and maxes).

  We used Dash to display multiple plotly graphs, where each graph is one of the above
pre-calculated data metrics. We then set each probe to be their own trace, which allowed us to
analyze the data more effectively by isolating or comparing certain probes on the graph. Each
of these probe traces has some hover text that provides more context and a link that
corresponds to their RIPE Atlas page. We configure it such that the X axis is time in
Coordinated Universal Time (UTC) and Y axis is latency (RTT) in ms.


### 1.3 Analysis and hypothesizes
  From our data, we see major spikes in latency from certain probes. Namely, Alaska and
Western Australia experience high latency. Generally, the other probes remained in a stable
range of 0 to 100 ms. We did some research on a few possible causes of this spike.

  Firstly, our research shows that there are 4 ground stations in Alaska and 4 in Western
Australia. The density of ground stations per unit area is lower in Alaska and Western Australia
than it is in the general areas of the other probes. This could be one explanation for the
unusually high latency, since the ground stations may not be equipped to handle high traffic. In
particular this is the case because of the way the routing for many of the starlink satellites work.
When the probe sends a request to a page on the internet, the user terminal it is connected to
on the ground sends a signal up onto the starlink network and is intercepted by the closest
satellite. If this satellite is a Starlink v1 satellite, then it will immediately proceed to re-downlink
the signal to the closest ground station. This means that for v1 satellites, it is important for there
to be a decent density of ground stations around the location of the probe, so when a v1 satellite
redirects the signal back down, it has stations ready to pick up the signal.

  Another possible explanation we found in our research could be the lack of satellites
above those regions. If there is not a high density of satellites above the probe’s user terminal,
there will not be as much bandwidth for communication, since there are less satellites to
intercept the signal. This effect is irregardless of what versions of the satellite are above the
probe. This is especially true for Alaska which is close enough to the polar region that there is a
very low concentration of satellites above it at any given time, meaning that for Alaska, this is
likely the more strongly contributing factor to the increased latency. However, in Western
Australia, there is a pretty high density of satellites over the probe at any given time, so it is
likely that Australia is being more affected by the lack of ground station density.

  In order to overcome these regional spikes in latency, SpaceX has a few options. One of
which it seems they are already beginning to implement. With the creation of later versions of
starlink, like the v2 satellites, the signal is instead sent between satellites using laser
communications before downlinking to a ground station. This allows for the network to optimize
at which position to send the signal before downlinking to a ground station. If there is a lack of
available ground stations around the user terminal, the v2 starlinks will send the signal closer
towards the destination before downlinking. As more v2 satellites are put into orbit, the problems
with ground station density will begin to disappear. This is important to implement because
currently in areas such as Africa and Asia there are little to no ground stations available,
meaning until robust laser communication is implemented with v2 satellites, these areas won’t be able to reliably join the starlink network. In fact, this is why these regions had no probes
available on the Ripe Atlas site.

  The option that would improve areas such as Alaska, would be to send more satellites
into orbit near the poles. This is less of an ideal solution, however, since getting satellites into
orbit around the poles due to launches being closer to the equator. Even if spacex expended the
additional resources to fill the poles with satellites, they will not last as long due to the increased
stress from the magnetic field at the poles. Finally, most of the regions on the poles typically
have a lower population meaning there is far less demand for starlink anyways, so places such
as alaska that have a higher population are an unfortunate byproduct of this situation.

  Going forward, it seems that SpaceX definitely has the potential and capability to not
only bring the internet to the entire globe, but to also serve as a possible replacement for cable,
since once the regional latency spikes are fixed, the sub-100ms RTTs of starlink would be
comparable to what is seen with cable already.
