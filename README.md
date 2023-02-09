# SwiftUI_AudioPlayerDemo


```swift
//
//  ContentView.swift
//  CustomAudioPlayerView
//
//  Created by paige shin on 2023/02/09.
//

import SwiftUI
import AVFoundation

struct ContentView: View {
    var body: some View {
        
        NavigationView {
            MusicPlayer()
                .navigationBarTitle("Music Player")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct MusicPlayer: View {
    
    @State var data: Data = .init(count: 0)
    @State var title: String = ""
    @State var player: AVAudioPlayer!
    @State var playing = false
    @State var width: CGFloat = 0
    @State var songs = ["black"]
    @State var currentIndex = 0
    @State var finish = false
    @State var delegate = AVDelegate()
    
    var body: some View {
        
        VStack(spacing: 20) {
            if let image = UIImage(data: self.data) {
                Image(uiImage: image)
                    .frame(width: 250, height: 250)
                    .background(Color.brown)
                    .cornerRadius(15)
            } else {
                Image(systemName: "music.note")
                    .frame(width: 250, height: 250)
                    .background(Color.brown)
                    .cornerRadius(15)
            }
            Text(self.title)
                .font(.title)
                .padding(.top)
            
            ZStack(alignment: .leading) {
                
                Capsule()
                    .fill(Color.black.opacity(0.08))
                    .frame(height: 8)
                
                Capsule()
                    .fill(Color.red)
                    .frame(width: self.width, height: 8)
                    .gesture(
                        DragGesture()
                            .onChanged({ value in
                                let x = value.location.x
                                self.width = x
                            })
                            .onEnded({ value in
                                let x = value.location.x
                                let screen = UIScreen.main.bounds.width - 30
                                let percent = x / screen
                                self.player.currentTime = percent * self.player.duration
                            })
                    )
                
            } //: ZSTACK
            .padding(.top)
            
            HStack(spacing: UIScreen.main.bounds.width / 5 - 30) {
                
                Button {
                    if self.currentIndex > 0 {
                        self.currentIndex -= 1
                        self.changeSongs()
                    }
                } label: {
                    Image(systemName: "backward.fill")
                        .font(.title)
                }
                
                Button {
                    let decrease = self.player.currentTime - 15
                    self.player.currentTime = decrease
                } label: {
                    Image(systemName: "gobackward.15")
                        .font(.title)
                }
                
                Button {
                    if self.player.isPlaying {
                        self.player.pause()
                        self.playing = false
                    } else {
                        if self.finish {
                            self.player.currentTime = 0
                            self.width = 0
                            self.finish = false
                        }
                        self.player.play()
                        self.playing = true
                    }
                } label: {
                    if self.playing && !self.finish {
                        Image(systemName: "pause.fill")
                            .font(.title)
                    } else {
                        Image(systemName: "play.fill")
                            .font(.title)
                    }
                    
                }
                
                Button {
                    let increase = self.player.currentTime + 15
                    if increase < self.player.duration {
                        self.player.currentTime = increase
                    }
                } label: {
                    Image(systemName: "goforward.15")
                        .font(.title)
                }
                
                Button {
                    if self.songs.count - 1 != self.currentIndex {
                        self.currentIndex += 1
                        self.changeSongs()
                    }
                } label: {
                    Image(systemName: "forward.fill")
                        .font(.title)
                }

                
            } //: HSTACK
            .padding(.top, 25)
            .foregroundColor(.black)
            
        }
        .onAppear {
            
            // MARK: INIT
            let url = Bundle.main.path(forResource: self.songs[self.currentIndex], ofType: "mp3")!
            self.player = try! AVAudioPlayer(contentsOf: URL(fileURLWithPath: url))
            self.player.delegate = self.delegate
            self.player.prepareToPlay()
            self.getData()
                     
            // MARK: UPDATE PROGRESS BAR
            Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
                if self.player.isPlaying {
                    let screen = UIScreen.main.bounds.width - 30
                    let value = self.player.currentTime / self.player.duration
                    print(self.player.currentTime)
                    self.width = screen * CGFloat(value)
                }
            }
            
            // MARK: RECEIVE
            NotificationCenter.default.addObserver(forName: NSNotification.Name("Finish"),
                                                   object: nil,
                                                   queue: .main) { _ in
                self.finish = true
            }
            
         
        }
        
        
    }
    
    // MARK: EXTRACT DATA FROM AVASSET
    func getData() {
        let asset = AVAsset(url: self.player.url!)
        
        
        for i in asset.commonMetadata {
            
            if i.commonKey?.rawValue == "artwork" {
                self.data = i.value as? Data ?? .init(count: 0)
            }
            
            if i.commonKey?.rawValue == "title" {
                self.title = (i.value as? String) ?? "No Title"
            }
            
        }
        
    }
    
    
    func changeSongs() {
        // MARK: INITIALIZE PLAYER
        let url = Bundle.main.path(forResource: self.songs[self.currentIndex], ofType: "mp3")!
        self.player = try! AVAudioPlayer(contentsOf: URL(fileURLWithPath: url))
        self.player.delegate = self.delegate
        self.player.prepareToPlay()
        self.getData()
        self.data = .init(count: 0)
        self.title = ""
        self.playing = true
        self.finish = false
        self.width = 0
        self.player.play()
    }
    
}

class AVDelegate: NSObject, AVAudioPlayerDelegate {
    
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        NotificationCenter.default.post(name: NSNotification.Name("Finish"), object: nil)
    }
    
}

```
