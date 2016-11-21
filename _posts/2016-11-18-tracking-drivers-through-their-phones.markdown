---
layout: post
title:  "Tracking Drivers with Bluetooth"
date:   2016-11-21 08:20:00 +0000
categories: privacy
---

Recently there has been a lot of noise around a plan by Transport for London to [track people around on the London Underground](http://www.theregister.co.uk/2016/11/17/tfl_to_track_tube_users_by_wifi_device_mac_address/) in order to work out journey patterns. The proposed system effectively works by capturing the MAC Addresses of Wi-Fi enabled devices as they pass through Underground stations. This has been loaded with controversy, with some activists on Twitter even vocalizing their plans to circumvent or disrupt the program.

[![Journey Time Smart Motorway Sign](/images/2016-11-18-tracking-drivers-through-their-phones/london-tweet.png)](https://twitter.com/DoubleJake/status/799668670418911232)

Regardless, this story reminded of Journey Time Monitoring technology. Despite the fact this technology is widespread, incredibly, there are very few people who know how it works.

## Journey Time Monitoring

![Journey Time Smart Motorway Sign](/images/2016-11-18-tracking-drivers-through-their-phones/26367617550_1ec8271ece_z.jpg)

*[Traffic Officer patrol on the M6 smart motorway - courtesy of Highways England](https://www.flickr.com/photos/highwaysengland/26367617550/in/album-72157665851298021/)*

Occasionally when driving along a motorway or major road, you may see a sign telling you the time it'll take you to drive to a given exit or destination. This can either be enabled during congestion, or sometimes when there are roadworks a portable display can be put up to indicate how long it will take to get through roadworks. To those of us who frequently drive on motorways (especially in Britain), this is nothing new. This system collects journey time data which can be useful in future planning, relaying this information to drivers via signs can potentially help with driver frustration. So how does this actually work?

### ANPR?

One way of doing this is ANPR (Automatic Number Plate Recognition). ANPR works to automatically recognize car registration plates using cameras and specialist software; these fixed cameras can then be used for law enforcement purposes (i.e. to check a car is being driven without road tax). By using these cameras in pairs, you can find out how long it takes a given car to get from point A to point B.

ANPR cameras do however require set-up and can be costly to install; especially in temporary applications.

### Bluetooth Journey Time Monitoring

Instead of ANPR, there is an alternative solution. Bluetooth traffic monitoring solutions allow for journey time to be calculated by capturing the MAC Addresses from Bluetooth enabled devices.

At Point A there is a roadside device that captures the MAC Addresses from Bluetooth devices, at Point B there is a device which does the same. If the same MAC Address is captured at Point A and Point B, the time difference between the captures is worked out - from there the journey time from Point A to Point B can be calculated. Outlier detections can then be removed, leading to the journey time data which can be relayed around.

Phones aren't the only bluetooth enabled devices in cars, indeed the cars themselves may be Bluetooth enabled, whether for hands-free calls or for internet-enabled services.

![Journey Time diagram](/images/2016-11-18-tracking-drivers-through-their-phones/bluetooth_speed_big.png)

*[Marketing material from Libelium](http://www.libelium.com/products/meshlium/smartphone-detection/)*

These sensors are relatively low-cost to install  and the road operator may well not even manage the data; often the vendor of the hardware will simply offer a web interface and API to access the data from a cloud web service. Such a cloud service may then be shared amongst the operators of multiple international road networks. Whilst it's easy at a technical level to identify where a given MAC Address is throughout the road network (covered by these sensors), I've not found an example of such a vendor actually doing this.

Bluetooth tracking isn't a new technology, in 2013 the City of London Corporation asked a company to stop [using recycling bins to track smartphone users](http://www.bbc.co.uk/news/technology-23665490) - but this technology has been widespread across the motorway network in Britain for many years with little controversy.

## Privacy Issues

Vendors of various MAC Address tracking devices claim they "encrypt" the sensor information passing through these devices. Unfortunately, in researching for this blogpost, I discovered that with most manufacturers this is really just MD5 hashing a MAC Address with no salt.

Hashing the MAC Addresses exists to prevent against one specific attack vector, identifying the MAC Address of every vehicle that drove past a given sensor. [Bluetooth MAC Addresses start with](https://www.lifewire.com/introduction-to-mac-addresses-817937) 6 characters which identify the manufacturer, then 6 characters unique to the device (e.g. MM:MM:MM:SS:SS:SS). For example, the [manufacturer prefix for BMW](http://www.coffer.com/mac_find/?string=BMW) is 00:01:A9, there are only 2238976116 potential MAC Addresses in this namespace. If the MAC Address was hashed using MD5 it would only take [about 4 minutes 15 seconds](http://calc.opensecurityresearch.com/) to iterate the entire MAC Address namespace of a given manufacturer and find the MAC Address of every BMW with Bluetooth that drove past a given sensor.

However, the real attack vector comes in the event someone wants to identify where a specific vehicle or smartphone user is within the road network. If you knew the MAC Address and access to the database of the Bluetooth Journey Time cloud service, you could essentially just calculate the MD5 hash of the MAC Address you wanted to locate and look for when it last appeared in the database. Worse, you could even set-up an alert to detect when a given set of MAC Addresses were picked up a Journey Time calculation device to find the location of a Bluetooth device.

## Conclusion

Journey Time tracking offers a great way to get data for planning of road capacity whilst also helping limit driver frustration; this may prove to be a lifesaving technology. Bluetooth Journey Time tracking allows this to be done cheaper than ever before, unfortunately this brings privacy questions as a result.
