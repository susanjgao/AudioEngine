//
//  CheckLatency.m
//  Latency
//  Created by Susan Gao on 11/13/13.
//

#import <Foundation/Foundation.h>
- (NSString*)calculatePlugInLatenciesInPatch:(AbstractNode *)node array:(NSMutableArray *)plugInArray
{
    NSString        *patchInfoText = @"";
    NSMutableArray  *pluginList = [NSMutableArray array];
    for (AbstractChannel *channel in node.channels)
    {
        float       sumLatency = 0.0;
        NSUInteger  numInsertSlots = [channel numUsedInsertSlotsForMIDIType:NO];
        NSString    *infoText = @"";
        for (NSUInteger slot = 0; slot < numInsertSlots; ++slot)
        {
            PlugIn *plugIn = [channel currentPlugInForSlot:slot isMIDISlot:NO];
            if (!plugIn)
                continue;
            float delay = ((Float64)plugIn.processDelay / AudioGetSampleRate()) * 1000;
            if (delay == 0.0)
                continue;
            sumLatency += delay;
            infoText = [infoText stringByAppendingString:[NSString localizedStringWithFormat:NSLocalizedString(@"  “%@” in Channel “%@”: %1.1fms\n", "Latency info display"), plugIn.shortName, channel.name, delay]];
            [pluginList addObject:plugIn];
        }
        // latency relevant for this channelstrip? => add all plugins in this channelstrip and the plugin infos
        if (sumLatency > [[NSUserDefaults standardUserDefaults] floatForKey:@"maxAcceptableLatency"])
        {
            [plugInArray addObjectsFromArray:pluginList];
            patchInfoText = [patchInfoText stringByAppendingString:infoText];
        }
    }
    return patchInfoText;
}

- (void)checkAudioLevels
{
    // check if the master is turned down, if you: warn the user that there is no sound…
    ConcreteChannel	*ch = self.document.mixerModel.masterChannel;
    if(ch.volume == 0)
    {
        if(!([[NSUserDefaults standardUserDefaults] boolForKey:@"dontWarnAgain_noSound"]))
        {
            Alert *alert = [Alert alertWithMessageText:NSLocalizedString(@"This concert will not produce any audio output until you adjust the master volume.",@"title of the alert")
                                             defaultButton:nil // will be "OK"
                                           alternateButton:nil
                                               otherButton:nil
                                 informativeTextWithFormat:@"%@",
                              NSLocalizedString(@"This is a precautionary measure to prevent potential feedback or loud audio from your speakers.",@"main text of the alert")];
            
            alert.alertStyle = NSWarningAlertStyle;
            alert.shoSuppressionButton = YES;
            [alert runModal];
            if (alert.suppressionButton.state == NSOnState)
                [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@"dontWarnAgain_noSound"];
        }
    }
    if(ch.isMuted)
    {
        if(!([[NSUserDefaults standardUserDefaults] boolForKey:@"dontWarnAgain_noSound"]))
        {
            Alert *alert = [Alert alertWithMessageText:NSLocalizedString(@"This concert will not produce any audio output until you change the master mute setting. ",@"title of the alert")
                                             defaultButton:nil // will be "OK"
                                           alternateButton:nil
                                               otherButton:nil
                                 informativeTextWithFormat:@"%@",
                              NSLocalizedString(@"This is a precautionary measure to prevent potential feedback or loud audio from your speakers.",@"main text of the alert")];
            
            alert.alertStyle = NSWarningAlertStyle;
            alert.shoSuppressionButton = YES;
            [alert runModal];
            if (alert.suppressionButton.state == NSOnState)
                [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@"dontWarnAgain_noSound"];
        }
    }
}

- (void)checkForLatency
{
    if([[NSUserDefaults standardUserDefaults] boolForKey:@"debugAutoDefaultAllAlerts"] || [[NSUserDefaults standardUserDefaults] boolForKey:@"dontWarnAgain_latency"])
        return;
    
    NSMutableArray  *latencyPlugInArray = [NSMutableArray array];    // array with all plugins that have a latency
    
    NSString    *latencyText = @"";
    
    // check concert level latency
    AbstractNode *concertNode = self.document.concert;
    NSString    *textPluginNames = [self calculatePlugInLatenciesInPatch:concertNode array:latencyPlugInArray];
    if (textPluginNames.length)
        latencyText = [[NSString stringWithFormat:NSLocalizedString(@"Concert: “%@”\n", "concert level latency"), concertNode.name] stringByAppendingString:textPluginNames];
        
        // iterate over all concert children
        for (AbstractNode *child in [(AbstractFolder *)concertNode children])
        {
            if (child.isSet)
            {
                // set level latency
                textPluginNames = [self calculatePlugInLatenciesInPatch:child array:latencyPlugInArray];
                if (textPluginNames.length)
                    latencyText = [latencyText stringByAppendingString:[[NSString stringWithFormat:NSLocalizedString(@"Set “%@”:\n", "set level latency"), child.name] stringByAppendingString:textPluginNames]];
                
                // latency of patches within the set
                for (AbstractNode *childPatch in [(AbstractFolder *)child children])
                {
                    if (childPatch.isPatch)
                    {
                        textPluginNames = [self calculatePlugInLatenciesInPatch:childPatch array:latencyPlugInArray];
                        if (textPluginNames.length)
                            latencyText = [latencyText stringByAppendingString:[[NSString stringWithFormat:NSLocalizedString(@"Patch “%@”:\n", "patch level latency"), (((AbstractNode *)childPatch).name)] stringByAppendingString:textPluginNames]];
                    }
                }
                
            } else if (child.isPatch)
            {
                // latency of patches within the concert
                textPluginNames = [self calculatePlugInLatenciesInPatch:child array:latencyPlugInArray];
                if (textPluginNames.length)
                    latencyText = [latencyText stringByAppendingString:[[NSString stringWithFormat:NSLocalizedString(@"Patch “%@”:\n", "patch level latency"), child.name] stringByAppendingString:textPluginNames]];
            }
        }
    
    // do we have an overall relevant latency?
    if(latencyText.length)
    {
        // only show the full text in authoring mode
        if(![[NSUserDefaults standardUserDefaults] boolForKey:@"patchAuthoringVersion"])
            latencyText = @"";
        
        Alert *alert = [Alert alertWithMessageText:NSLocalizedString(@"This concert contains plugins that add output latency.  This will cause a longer delay between audio or MIDI input and audio output.", @"Title for latency alert window")
                                         defaultButton:NSLocalizedString(@"Ignore", @"Ignore button")
                                       alternateButton:NSLocalizedString(@"Bypass All", @"Bypass button")
                                           otherButton:nil
                             informativeTextWithFormat:@"%@", latencyText];
        alert.alertStyle = NSCriticalAlertStyle;
        alert.shoSuppressionButton = YES;
        
        NSModalResponse button = [alert runModal];
        if (alert.suppressionButton.state == NSOnState)
            [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@"dontWarnAgain_latency"];
        
        if (button == NSAlertSecondButtonReturn)
        {
            for (PlugIn *plugIn in latencyPlugInArray)
                plugIn.bypassed = YES;
        }
    }
}
