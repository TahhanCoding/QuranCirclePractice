

# Quran Circle Practice

This file contains my work in Quran Circle iOS Application. Quran Circle uses Adaptive Learning to create your daily plans. The more you use it, the better it understands your habits and learning style. Your lessons will continuously evolve and adapt to your progress and individual needs, just like they would with a teacher. check it on the Appstore: [Quran Circle](https://apps.apple.com/us/app/qc-memorize-quran-learn-ayat/id1499829628).

I have managed to add bunch of features and fix many issues, from modifying storyboard constraints, fixing UI elements and enhancing app performance and user experience.

## Tasks: 

1. Localization of strings with/out string arguments.
2. Localization of strings with Integer arguments.
3. Fix navigation bar Title UI.
4. Fix audio player memory leak and Modify downloading scenario. 
5. Preparing Assets in Photoshop.


### 1. Localization of strings with/out string arguments

The localization is done  in two files: 
* **Localizable.strings** for normal strings that doesn't have Plural format.
* **Localizable.stringsdict** for strings that has a counter variable and needs to be formatted in a convenient way for each plural form in each language. 

### Initial steps: 
1- Add required languages in the bundle info.
2- Add .strings file in the most upper folder in the project and name it "**Localizable**".
3- From File Inspector click Localize and select languages to be supported. 

### Hint:
* If you read about using **genstring** in the terminal, and met problems using it. I will make the road shorter for you. genstring can generate strings for the tableName existed in the folder you are in **only**. 
* So, for every folder and subFolder in your project you will need a different tableName. you might try this approach **if you have some files that have huge number of strings that needs to be localized. so, it's easier to use genstring to automatically generate keys for you.** If number of strings isn't that much, you can use only one Localizable.strings and write keys manually.
* You may want to use it for files like that, and then manually **collect all strings in Localizable.strings**.
    

### Localization the normal way:

```
let name = NSLocalizedString("Ahmed", tableName: "Localizable", comment: "")
let message = NSLocalizedString("Message", tableName: "Localizable", comment: "")
let fmtMessage = String(format: message, arguments: name)
```
- It's a long way to localize all strings in a project this way. Instead do the following:

Create a new Swift file and name it Extensions, then add the following code:

```
extension String {
      var localized: String {
        NSLocalizedString(self, tableName: "Localizable", comment: "")
  }
      func localizedWithArguments(_ arguments: CVarArg...) -> String {
        String(format: self.localized(), arguments: arguments)
  }
}
```
* Both Functions have **tableName: "Localizable"** inserted, if you are using a different tableName edit it. 
* If you are willing to use multiple .strings files, make localized a func instead of computed variable and add a paramater (tableName: String) to both functions, then use it.

Then to localize the same strings: 
```
let name = "Ahmed".localized
let message = "Message".localizedWithArguments(name)
```

### Localizable.strings(en):

- "Message" is the key and other side is the value.
- %@ is a place holder for a string argument.
- Comments are used to help translators.

```
/*No comment provided*/
"Message" = "Hello %@ !"; 

/*No comment provided*/
"Ahmed" = "Ahmed";
```

### Localizable.strings(ar):

```
/*No comment provided*/
"Message" = "مرحبًا %@ !"; 

/*No comment provided*/
"Ahmed" = "أحمد"
```
###  Results
```
let name = "Ahmed".localized()
let message = "Message".localizedWithArguments(name)
```
#### In a SwiftUI View add:
```
Text(message) 
```
In **en Scheme** it shows:

```
							   Hello Ahmed !
```

In **ar Scheme** it shows:

```
							   ! مرحبًا أحمد
```

### 2. Localization of strings with Integer arguments.

```
let lblText = "lbl.tag = 1".localizedWithArguments(count, suraName)
```

#### stringsdict Implementation


![4c478839-c381-4144-880a-d1d77d34a20d](https://user-images.githubusercontent.com/97001250/227047319-77070374-6172-4426-bae2-1719c5dfa9b1.jpg)




### 3. Fix navigation bar Title UI

 I noticed that when navigation bar title exceeds 35 chars it tends to shift alignment. So I came with this fix. 

``` 
// setting default alignment
titleLabel.textAlignment = .center


if title.count > 35 {
let appLang = Locale.preferredLanguages[0]

if appLang == "ar" { titleLabel.textAlignment = .right }
else		   { titleLabel.textAlignment = .left  }

//I usually use “Hello” as a marker to filter console messages.
print("Hello long title: fix your position please")

}
``` 

<img width="606" alt="Screenshot 2023-03-21 at 3 13 13 PM" src="https://user-images.githubusercontent.com/97001250/226616681-03d9b529-97b8-49b3-ae8e-4fa36e912768.png">


### 4. Fix audio player memory leak and Modify downloading scenario

The issue with the audio player is that after looping some short ayahs, 3 or 4 times, **player starts to skip an ayah without playing it**, although it was shown in the console that url is passed correctly to the player.

The original code had initiated the AVPlayer in the same function that retrieved the track URL, which **made it challenging to debug and identify the cause of the playback issue**. To address the issue, I chose to **break down the code into smaller, independent components through decoupling**. This gave me the ability to **maintain each part of the code individually** and solve the problem more effectively.

Making an abstract with a recite func that can be used to recite a url, whatever this "recite" means or do.
```
protocol AVPlayerProtocol {
	func recite(_ url: URL)
}
```

 **Implementation of recite function:**
  
```
class  MyAVPlayer: AVPlayerProtocol {

	static let shared = MyAVPlayer()

	// AVQueuePlayer has better memory management
	private var  player = AVQueuePlayer()
	
	// Public, so when player Item play till end, Next button is initiated
	public var  playerItem: AVPlayerItem!

	func recite(_ url: URL) {

		// empty memory used by player to reduce memory usage
		playerItem = nil
		player.removeAllItems()

		playerItem = AVPlayerItem(url: url)
		player = AVQueuePlayer(playerItem: self.playerItem)

		player.currentItem?.preferredForwardBufferDuration = TimeInterval(1.5)
		player.automaticallyWaitsToMinimizeStalling = player.currentItem?.isPlaybackBufferEmpty ?? false
		player.seek(to: CMTime.zero)

		player.play()

		print("Hello: audioPath is \(url)")
	}
}
```
AyaPlayer class that has a property of AVPlayerProtocol type, and AyaPlayer itself is **initiated with a shared instance of MyAVPlayer**.
```
class  AyaPlayer {

	static  let  shared = AyaPlayer(myPlayer: MyAVPlayer.shared)

	private let  myPlayer: AVPlayerProtocol

	init(myPlayer: AVPlayerProtocol) {
		self.myPlayer = myPlayer
	}

	func reciteAya(_ url: URL) {
		myPlayer.recite(url)
	}

}
```

Then the main func **passes the url to the shared instance of AyaPlayer**:

- Audio was being played remotely even in loops which **consumes data**, and tracks are downloaded only when the user closes/reopen the app while connected to the internet. I **suggested modifying this to a more user friendly scenario, when user plays a track, It is -and related Items- downloaded directly and shown in the downloads screen.** 


```
func  reciteAya()  {	
	// path here is optional string.
	let path = DownloadHelper.shared.haveLocalFile(quranItem: quranItems[index], reciter: reciter)

	if path == nil {
	// If file doesn't exist download it and its related items.
	DownloadHelper.shared.downloadAllFiles(sura: quranItems[index].sura, reciterID: reciter.id, quran: quranItems)
	}

	let audioURL: URL
	if let path = path { audioURL = URL(fileURLWithPath: path) } 
	else { 
		// If locale file doesn't exist, play remote file 
		// At the very first play, remote file is played
		// downloading in the background, then plays locally next time.
		if  let remoteURL = URL(string: AudioHelper.shared.getURLStrFromQuran(quranItem: quranItems[index],reciter: reciter)) {
			audioURL = remoteURL
		} else { return }
}

	AyaPlayer.shared.reciteAya(audioURL)

	NotificationCenter.default.addObserver(self,selector: #selector(self.playerDidFinishPlaying(sender:)),name: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: MyAVPlayer.shared.playerItem)
}

```

After doing this, issue was reduced from happening after 3 loops to around 13 loops as avplayer memory is handled very clearly, but issue wasn't solved completely.

Debugging and logically thinking why a track is skipped? why next button is initiated instantly? So it might be a problem with the observer of playing item till end. Happens that **removeObserver was written at the deinit { }** at the view global scope. And **observers aren't cleared while the view is live** and are accumulated which caused memroy issue that **sends false next action**. So I decided to **move removeObserver right before the addObserver. now everytime, old observer is removed from memory and new one is initiated and problem was solved.**

```
NotificationCenter.default.removeObserver(self, name: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: nil)

NotificationCenter.default.addObserver(self,selector: #selector(self.playerDidFinishPlaying(sender:)),name: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: MyAVPlayer.shared.playerItem)

```


### 5. Preparing Assets in Photoshop

Finally I used photoshop which I have basic knowledge of it, to prepare images to use in the intro section. 



<img width="1013" alt="Screenshot 2023-03-17 at 11 04 33 PM" src="https://user-images.githubusercontent.com/97001250/226616414-1e44124b-2e78-4612-92d0-fea3cf6e1d85.png">

            
The App was completed and updated on the Appstore. 

### Alhamdulillah - الحمد لله. 💚

