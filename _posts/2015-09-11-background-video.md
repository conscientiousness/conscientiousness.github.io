---
layout: post
title: 'Object-C 學習筆記 - 背景影片'
date: 2015-09-11 07:15
author:     "Jesse"
catalog:    false
tags:
    - Objc
    - iOS
---
實現背景影片連續播放，簡略步驟
- 先設定AVAudioSession避免背景播放
- init AVPlayer並給予影片URL路徑
- 實體化AVPlayerLayer給予剛才設定好的AVPlaye
- 設定影片的比例，`AVLayerVideoGravityResizeAspectFill`佔滿螢幕且不失比例
- 設定layer的大小，佔滿全螢幕
- 然後add到storyboard拉的view的layer上，但要注意新增的位置（[可參考](http://jesse.logdown.com/posts/297518-view)）
- 剩下就設定AVPlayer，用Notification實現連續播放



```objc
#import <AVFoundation/AVFoundation.h>

- (void)viewDidLoad {
    [super viewDidLoad];

    //Not affecting background music playing
    NSError *sessionError = nil;
    [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryAmbient error:&sessionError];
    [[AVAudioSession sharedInstance] setActive:YES error:&sessionError];

    //Set up player
    NSURL *movieURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"login-background-video" ofType:@"mp4"]];
    AVAsset *avAsset = [AVAsset assetWithURL:movieURL];
    AVPlayerItem *avPlayerItem =[[AVPlayerItem alloc]initWithAsset:avAsset];
    self.avplayer = [[AVPlayer alloc]initWithPlayerItem:avPlayerItem];
    AVPlayerLayer *avPlayerLayer =[AVPlayerLayer playerLayerWithPlayer:self.avplayer];
    [avPlayerLayer setVideoGravity:AVLayerVideoGravityResizeAspectFill];
    [avPlayerLayer setFrame:[[UIScreen mainScreen] bounds]];
    [self.videoView.layer addSublayer:avPlayerLayer];

    //Config player
    [self.avplayer seekToTime:kCMTimeZero];
    [self.avplayer setVolume:0.0f];
    [self.avplayer setActionAtItemEnd:AVPlayerActionAtItemEndNone];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(playerItemDidReachEnd:)
                                                 name:AVPlayerItemDidPlayToEndTimeNotification
                                               object:[self.avplayer currentItem]];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(playerStartPlaying)
                                                 name:UIApplicationDidBecomeActiveNotification object:nil];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    [self.avplayer pause];
}

- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    [self.avplayer play];
}

- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self name:AVPlayerItemDidPlayToEndTimeNotification object:nil];
}

- (void)playerItemDidReachEnd:(NSNotification *)notification {
    AVPlayerItem *p = [notification object];
    [p seekToTime:kCMTimeZero];
}

- (void)playerStartPlaying
{
    [self.avplayer play];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (IBAction)exitBtnPressed:(id)sender {
    [self dismissViewControllerAnimated:true completion:nil];
}
@end

```