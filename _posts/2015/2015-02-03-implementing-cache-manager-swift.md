---
title: "Implementing a cache manager in Swift"
date: "2015-02-03"
categories: 
  - "technical"
tags: 
  - "cache"
  - "manager"
  - "nscoding"
  - "singleton"
  - "swift"
---

In this article we'll see how we can make a simple cache manager in swift.

The goal is to store NSCoding compliant objects to the phone and restore them afterwards. We'll use an identifier string that is provided to the "save" function to generate unique filename to save the object on the phone.

The CacheManager implementation is pretty straightforward

- "sharedInstance" : a computed property  for the singleton
- "filenameFromIdDico" property : a dictionary used to track the unique filename associated to an object identifier
- "objectMaxId" : a private property that represent the current object maximum id
- "init()" and "encodeWithCoder()" : the methods to be NSCoding compliant
- "saveObject()" and "loadObject()" methods to manipulate the objects we want to cache
- "saveToDevice()" : a  method used to save the CacheManager

 

First, let's declare our class. The objectMaxId variable will be used to generate the next unique filename to link for an object identifier. You could use "filenameFromIdDico.count" but by using another variable you can improve more easily your CacheManager. For example you could allow for removal of objects and still guarantee that you generate a valid unique filename.

class CacheManager: NSObject, NSCoding {
	private var objectMaxId: Int = 0
	private var filenameFromIdDico: \[String : String\] = \[:\]

	override init(){
		super.init()
	}	
}

 

First of all, here is an utility method that we are going to use to generate a full path from a filename

class func pathInDocDirectory(filename: String)->String? {
	let paths = NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory.DocumentDirectory, NSSearchPathDomainMask.UserDomainMask, true)
	if paths.count > 0 {
		if let path: String = paths\[0\] as? String {
			return path + "/" + filename
		}
	}
	return nil
}

 

Since Swift does not currently allow class to have static variable, the singleton uses the method of the computed property with a Struct containing a static variable (check this [singleton tutorial](http://thatthinginswift.com/singletons/) to learn more about this). The only difference is that we try to load an existing CacheManager from the phone instead of simply instantiating a new cache manager

class var sharedInstance: CacheManager {
	struct Static {
		static var instance: CacheManager?
		static var token: dispatch\_once\_t = 0
	}
	dispatch\_once(&Static.token) {
		//we try to load an existing CacheManager, otherwise we create a new one
		if let filepath = CacheManager.pathInDocDirectory(CACHE\_MANAGER\_NAME) {
			if let mgr = NSKeyedUnarchiver.unarchiveObjectWithFile(filepath) as? CacheManager{
				Static.instance = mgr
			}
		}
		if (Static.instance == nil) {
			Static.instance = CacheManager()
		}
	}
	return Static.instance!
}

 

The init method is used by the NSKeyedUnarchiver.unarchiveObjectWithFile when we reload the CacheManager

required init(coder aDecoder: NSCoder) {
	super.init()
	self.objectMaxId = aDecoder.decodeIntegerForKey("objectMaxId")
	if let dico:\[String:String\] = aDecoder.decodeObjectForKey("filenameFromUrlDic") as? \[String:String\] {
		self.filenameFromIdDico = dico
	}
}

 

The encodeWithCoder is used by the NSKeyedArchiver.archiveRootObject to save the CacheManager

func encodeWithCoder(aCoder: NSCoder) {
	aCoder.encodeInteger(self.objectMaxId, forKey: "objectMaxId")
	aCoder.encodeObject(self.filenameFromIdDico, forKey: "filenameFromUrlDic")
}

 

Now the function to save an object. Of course the objects must either be one of those provided by Apple (Int, String, Array, Dictionnary, ...) or one of your object that is NSCoding compliant.

If it's an object whose ID we do not know, we assign it a unique filename, then we generate the filepath to save it. It's important to note that we do not track the complete filepath because it contain a unique ID that changes each time you build your app. All this process is done inside a synchronised block for thread safety reasons; we do not want 2 threads to increment the "objectMaxId" field before generating the filename.

Every time we save an object, we save the CacheManager.

func saveObject(object:AnyObject, identifier:String) -> Bool {
	if (identifier.isEmpty) {
		return false
	}
	
	//we sync on the object to be sure that only thread at a time generates a new objectId
	objc\_sync\_enter(self)
	var filename: String
	//we check to see if the CacheManager has a caches object for this identifier
	if let filenameFromDico: String = self.filenameFromIdDico\[identifier\] {
		filename = filenameFromDico
	}
	else {
		self.objectMaxId++
		filename = "object." + String(self.objectMaxId)
		self.filenameFromIdDico\[identifier\] = filename
	}
	objc\_sync\_exit(self)
	
	var status: Bool = false
	//we generate the full path for the object every time instead of caching it because the path contains a unique identifier that changes with each build, so we mustn't cache it
	if let filepath: String = CacheManager.pathInDocDirectory(filename) {
		NSKeyedArchiver.archiveRootObject(object, toFile: filepath)
		status = true
	}
	self.saveToDevice()
	return status;
}

 

And finaly the method to load an object from its identifier. If we find it in the dictionnary, we load this object

func loadObject(identifier:String) -> AnyObject? {
	if let filename: String = self.filenameFromIdDico\[identifier\] {
		if let filepath = CacheManager.pathInDocDirectory(filename) {
			return NSKeyedUnarchiver.unarchiveObjectWithFile(filepath)
		}
	}
	return nil
}

 

Be sure to download the demo of this [cache manager on github](https://github.com/3IE/swift-cache-manager) !
