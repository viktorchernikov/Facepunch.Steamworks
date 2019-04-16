# Facepunch.Steamworks
Another fucking c# Steamworks implementation

[![Build Status](http://build.facepunch.com/buildStatus/icon?job=Facepunch/Facepunch.Steamworks/master)](http://build.facepunch.com/job/Facepunch/job/Facepunch.Steamworks/job/master/)

## Why

The Steamworks C# implementations I found that were compatible with Unity have worked for a long time. But I hate them all. For a number of different reasons.

* They're not C#, they're just a collection of functions.
* They're not up to date.
* They require a 3rd party native dll.
* They can't be compiled into a standalone dll (in Unity).
* They have a license.

C# is meant to make things easier. So lets try to wrap it up in a way that makes it all easier.

## What

### Get your own information

```csharp
SteamClient.SteamId // Your SteamId
SteamClient.Name // Your Name
```

### View your friends list

```csharp
foreach ( var friend in SteamFriends.GetFriends() )
{
    Console.WriteLine( "{friend.Id}: {friend.Name}" );
    Console.WriteLine( "{friend.IsOnline} / {friend.SteamLevel}" );
    
    friend.SendMessage( "Hello Friend" );
}
```

But what if you want to get a list of friends playing the same game that we're playing?

```csharp
foreach ( var friend in client.Friends.All.Where( x => x.IsPlayingThisGame ) )
{
    // 
}
```

### App Info

```csharp
Console.WriteLine( SteamApps.GameLanguage ); // Print the current game language
var installDir = SteamApps.AppInstallDir( 4000 ); // Get the path to the Garry's Mod install folder

var fileinfo = await SteamApps.GetFileDetailsAsync( "hl2.exe" ); // async get file details
DoSomething( fileinfo.SizeInBytes, fileinfo.Sha1 );
```

### Get Avatars

```csharp
var image = await SteamFriends.GetLargeAvatarAsync( steamid );
if ( !image.HasValue ) return DefaultImage;

return MakeTextureFromRGBA( image.Data, image.Width, image.Height );
```

### Get a list of servers

```csharp
using ( var list = new ServerList.Internet() )
{
    list.AddFilter( "map", "de_dust" );
    await list.RunQueryAsync();

    foreach ( var server in list.Responsive )
    {
        Console.WriteLine( $"{server.Address} {server.Name}" );
    }
}
```

### Achievements

List them

```csharp
foreach ( var a in SteamUserStats.Achievements )
{
    Console.WriteLine( $"{a.Name} ({a.State}})" );
}	
```

Unlock them

```csharp
var ach = new Achievement( "GM_PLAYED_WITH_GARRY" );
ach.Trigger();
```

### Voice

```csharp

SteamUser.VoiceRecord = KeyDown( "V" );

if ( SteamUser.HasVoiceData )
{
    var bytesrwritten = SteamUser.ReadVoiceData( stream );
    // Send Stream Data To Server or Something
}

```


### Auth

```csharp

// Client sends ticket data to server somehow
var ticket = SteamUser.GetAuthSessionTicket();


// server listens to event
SteamServer.OnValidateAuthTicketResponse += ( steamid, ownerid, rsponse ) =>
{
    if ( rsponse == AuthResponse.OK )
        TellUserTheyCanBeOnServer( steamid );
    else
        KickUser( steamid );
};

// server gets ticket data from client, calls this function.. which either returns
// false straight away, or will issue a TicketResponse.
if ( !SteamServer.BeginAuthSession( ticketData, clientSteamId ) )
{
    KickUser( clientSteamId );
}

//
// Client is leaving, cancels their ticket OnValidateAuth is called on the server again
// this time with AuthResponse.AuthTicketCanceled
//
ticket.Cancel();

```

### Utils

```csharp
SteamUtils.SecondsSinceAppActive;
SteamUtils.SecondsSinceComputerActive;
SteamUtils.IpCountry;
SteamUtils.UsingBatteryPower;
SteamUtils.CurrentBatteryPower;
SteamUtils.AppId;
SteamUtils.IsOverlayEnabled;
SteamUtils.IsSteamRunningInVR;
SteamUtils.IsSteamInBigPictureMode;
```

# Usage

## Client

To initialize a client you can do this.

```csharp
using Steamworks;

// ...

try 
{
    SteamClient.Init( 4000 );
}
catch ( System.Exception e )
{
    // Couldn't init for some reason (steam is closed etc)
}
```

Replace 4000 with the appid of your game. You shouldn't call any Steam functions before you initialize.

When you're done, when you're closing your game, just shutdown.

```csharp
SteamClient.Shutdown();
```

## Server

To create a server do this.

```csharp
var serverInit = new SteamServerInit( "gmod", "Garry Mode" )
{
    GamePort = 28015,
    Secure = true,
    QueryPort = 28016
};

try
{
    Steamworks.SteamServer.Init( 4000, serverInit );
}
catch ( System.Exception )
{
    // Couldn't init for some reason (dll errors, blocked ports)
}
```

# Help

Wanna help? Go for it, pull requests, bug reports, yes, do it.

You can also hit up the [Steamworks Thread](http://steamcommunity.com/groups/steamworks/discussions/0/1319961618833314524/) for help/discussion.

# License

MIT - do whatever you want.
