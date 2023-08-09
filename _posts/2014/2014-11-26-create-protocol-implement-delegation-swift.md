---
title: "Create protocol and implement delegation in Swift"
date: "2014-11-26"
categories: 
  - "technical"
tags: 
  - "swift"
---

First of all, start by creating a new Swift file and name it something like _HLRequestDelegate_ ; that will define your protocol.

```swift
protocol HLRequestDelegate {
    
    func doNothing();
    func request(request: HLRequest, didSucceedWithData: NSData);
}
```

To call your delegate, you may now create a new class instance and call methods defined in previous Swift file.

```swift
class HLRequest: NSObject, NSURLConnectionDelegate {
    
    var delegate: HLRequestDelegate? // create a new delegate instance
    
    func executeRequest()
    {
        println("Attempt to execute request")
        delegate?.request(self, didSucceedWithData: NSData()) // let's trigger our delegate instance
    }
}
```

The last thing to do is to implement our delegate. Open your Swift class and add your protocol at the top of class declaration, and implement the methods, this way:

```swift
class EYHomeVC: UIViewController, HLRequestDelegate[/objc]

[objc]func doNothing() {
    println("I'm totally useless");
}

func request(request: HLRequest, didSucceedWithData data: NSData)
{
    var stringData: NSString = NSString(data: NSData(data: data), encoding: 4)
    println("Response body: \\(stringData)")
}
```

You can now get your methods called by assigning delegate:

```swift
var productRequest: HLRequest = HLRequest()
productRequest.delegate = self
productRequest.executeRequest()
```

Find [complete snippet here](https://github.com/fiftydegrees/swift-snippets).
<br>
<br>

---------------------------------------
<br>
Auteur: **herve.droit**
