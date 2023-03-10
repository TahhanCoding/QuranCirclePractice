# QuranCirclePractice
##  written by Ahmed AlTahhan 

This project contains functions and demo exercise for my work in Quran Circle iOS Application.

The localization is done generally in two files: 
* Localizable.strings for normal strings that doesn't have Plural format.
* Localizable.stringsdict for strings that has a counter variable and needs to be formatted in a convenient way for each plural form in each language. 

1- Add languages you want in the bundle info.
2- Add .strings file in the most upper folder in the project and name it "Localizable".
3- From File Inspector click Localize and select languages that the file supports.


```
extension String {
  // .localized() function determines a link between the string to localize and its key in the .strings or .stringsdict file. here, Localizable.strings
      func localized() -> String {
        NSLocalizedString(self, tableName: "Localizable", comment: "")
  }
  
  // .localizedWithArguments() function takes the arguments to format string with if any.
      func localizedWithArguments(_ arguments: CVarArg...) -> String {
        String(format: self.localized(), arguments: arguments)
  }
}

```

* Both Functions have tableName: "Localizable" inserted, if you are using a different tableName edit it. If you are willing to use multiple .strings files with differenet names, add a paramater (tableName: String) to both functions, then insert the name .localized(tableName: "name")

#Hint about using genstring
* If you read about using genstring, and met problems using it. I will make the road shorter for you. genstring can generate strings for the tableName existed in the folder you are in only.
    so, for every folder and subFolder in your project you will need a different tableName. you might try this approach if you have some files that have huge number of strings that needs to be localized. so, it's easier to use genstring. If number of strings isn't that much, you can use only one Localizable.strings and write keys manually.
    

Localizable.strings (English):

```
/*"Message" is the key and other side is the value */
"Message" = "Hello %@"; 

/*No comment provided, comments are used to help translator*/
"Ahmed" = "Ahmed";
```

Localizable.strings (Arabic):

```
/*"Message" is the key and other side is the value */
"Message" = "???????????? %@"; 

/*No comment provided*/
"Ahmed" = "????????"
```

```
let name = "Ahmed".localized()
let message = "Message".localizedWithArguments(name)
Text(message) in a SwiftUI View 
if en: Hello Ahmed
if ar: ???????????? ????????

```

## AVPlayer

``` 
import AVFoundation
import Foundation

protocol AVPlayerProtocol {
    func play(url: URL)
}

class MyAVPlayer: NSObject, AVPlayerProtocol {

    private var player: AVPlayer!
    private var urlAsset: AVURLAsset!
    
    // public because it used to observe play end to initiate button next
    public  var playerItem: AVPlayerItem!
    
        func play(url: URL) {
            
            // Clean up the previous player and item if any (Tested (prnt) and Working)
            player?.pause()
            player = nil
            playerItem = nil
            urlAsset = nil
            // Observe the player item's status to ensure it's ready to play
            playerItem?.addObserver(self, forKeyPath: #keyPath(AVPlayerItem.status), options: [.old, .new], context: nil)

            // Create a new player item and asset
            urlAsset = AVURLAsset(url: url, options: nil)
            playerItem = AVPlayerItem(asset: urlAsset!)
            player = AVPlayer(playerItem: playerItem)
            player?.currentItem?.preferredForwardBufferDuration = TimeInterval(3)
            player?.automaticallyWaitsToMinimizeStalling = player?.currentItem?.isPlaybackBufferEmpty ?? false
            player?.seek(to: CMTime.zero)
            player?.play()
            print("HELLO: playing  URL: \(url)")
        }

        override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
            if keyPath == #keyPath(AVPlayerItem.status) {
                let status = change?[.newKey] as? AVPlayerItem.Status
                if status == .readyToPlay {
                    player?.play()
                }
            }
        }
        
        deinit {
            playerItem?.removeObserver(self, forKeyPath: #keyPath(AVPlayerItem.status))
            print("HELLO: Memory is now Free of AVPlayer")
        }
}


class AyaPlayer {
    
    let myPlayer: AVPlayerProtocol
    
    init(myPlayer: AVPlayerProtocol) {
        self.myPlayer = myPlayer
    }
    
    func playAya(_ url: URL) {
        myPlayer.play(url: url)
    }
}
```
to use the class:
```
//     let myPlayer = MyAVPlayer()
//     let ayaPlayer = AyaPlayer(myPlayer: myPlayer)
```

## Thread-Safe DownloadHelper Class
```
class DownloadHelper {

    private let queue = DispatchQueue(label: "downloadHelperThread")

   func doSomethingProcessTimeConsuming() {  


     queue.async {    [weak self] in 
     // do something
     
     {
```

