# OpenHAB integration of itunes-API

## Requirements

* MacOS with iTunes
* itunes-api - https://github.com/eiGelbGeek/itunes-api
* openhab2
*  -> Javascript Transformation
*  -> HTTP Binding
*  -> Expire Binding
* JQ - https://github.com/stedolan/jq
*  -> sudo apt-get install jq

### Installation

Copy all files to Openhab2 based on folder structure!

Create all necessary items, rules and the sitemap.

Fill in the IP address in the HTML file

Complete the configuration of the two bash scripts.

* -> sudo chown openhab:openhab /etc/openhab2/scripts/\*.sh
* -> sudo chmod +u+x /etc/openhab2/scripts/\*.sh
* -> sudo chown openhab:openhab /etc/openhab2/transform/\*.js
* -> sudo chown openhab:openhab /etc/openhab2/html/\*.html

Before you update the playlists, make a back-up!

I've tested the script several times, but I'm not responsible for data loss!

### Sitemap

```js
Webview url="/static/itunes_cover.html" height=8

Text item=iTunes_Artist label="Artist [%s]"
Text item=iTunes_Title label="Titel [%s]"
Text item=iTunes_Album label="Album [%s]"
Text item=iTunes_Playlist label="Album [%s]"
Text item=iTunes_Player_State label="Player State [%s]"

Selection item=playlist_selection label="Playliste" icon="playlist" mappings=[0="Playlisten Updaten"]
Switch item=webradio_delete "Delete Webradiostations from Library"
Selection item=webradio_selection label="Webradio" icon="playlist" mappings=[0="Choose Webradiostation", 1="YourFavoriteSation-1", 2="YourFavoriteSation-2", 3="YourFavoriteSation-3", 4="YourFavoriteSation-4"]
Switch item=Airplay_Office label="Airplay Office"
Switch item=Airplay_Office_Status label="Airplay Office Status"
Slider item=Volume_Main label="Main Volume"
Slider item=Volume_Office label="Volume Office"
Switch item=iTunes_Play label="Play"
Switch item=iTunes_PlayPause label="Play/Pause"
Switch item=iTunes_Pause label="Pause"
Switch item=iTunes_Stop label="Stop"
Switch item=iTunes_Previous label="Previous"
Switch item=iTunes_Next label="Next"
Switch item=iTunes_Mute label="Mute"
Selection item=Shuffle_Item label="Shuffle" mappings=[0="Off", 1="Songs", 2="Albums", 3="Groupings"]
Selection item=Repeat_Item label="Repeat" mappings=[0="Off", 1="One", 2="All"]
```
### items

```js
String iTunes_Artist "Artist [%s]" { http="<[http://XXX.XXX.XXX.XXX:8181/now_playing:6000:JS(itunes_artist.js)]" }
String iTunes_Title "Titel [%s]" { http="<[http://XXX.XXX.XXX.XXX:8181/now_playing:6000:JS(itunes_title.js)]" }
String iTunes_Album "Album [%s]" { http="<[http://XXX.XXX.XXX.XXX:8181/now_playing:6000:JS(itunes_album.js)]" }
String iTunes_Playlist "Playlist [%s]" { http="<[http://XXX.XXX.XXX.XXX:8181/now_playing:6000:JS(itunes_playlist.js)]" }
String iTunes_Player_State "Player State [%s]" { http="<[http://XXX.XXX.XXX.XXX:8181/now_playing:6000:JS(itunes_player_state.js)]" }

Number playlist_selection "Play Playlist" { expire="5s,command=0" }
Number webradio_selection "Play Webradiostation" { expire="5s,command=0" }
Switch webradio_delete "Delete Webradiostations from Library" { expire="5s,command=OFF" }
Switch Airplay_Office "Airplay Office"
Switch Airplay_Office_Status "Airplay Office Status"
Dimmer Volume_Main "Main Volume [%s]"
Dimmer Volume_Office "Volume Office [%s]"
Switch iTunes_Play "Play" { expire="1s,command=OFF" }
Switch iTunes_PlayPause "Play/Pause"
Switch iTunes_Pause "Pause" { expire="1s,command=OFF" }
Switch iTunes_Stop "Stop" { expire="1s,command=OFF" }
Switch iTunes_Previous "Previous" { expire="1s,command=OFF" }
Switch iTunes_Next "Next" { expire="1s,command=OFF" }
Switch iTunes_Mute "Mute"
Number Shuffle_Item
Number Repeat_Item

```
### Rule

```js
rule"Play Playlist"
when
  Item playlist_selection received update
then
  if (playlist_selection.state =! 0) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "playlist " + playlist_selection.state)
end
```

```js
rule"Delete Webradiostations from Library"
when
  Item webradio_delete changed from OFF to ON
then
  executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "webradio " + "delete")
end
```

```js
rule"Play Webradiostation"
when
  Item webradio_selection received update
then
  if (webradio_selection.state =! 0) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "webradio " + webradio_selection.state)
end
```

```js

rule"Airplay ON/OFF"
when
  Item Airplay_Office received update
then
  if (Airplay_Office.state = ON) {
    executeCommandLine("/etc/openhab2/scripts/iTunes.sh " + "airplay_on " + "ID_FROM_AIRPLAY_DEVICE" + " Airplay_Office" + " Airplay_Office_Staus")
  } else {
    executeCommandLine("/etc/openhab2/scripts/iTunes.sh " + "airplay_off " + "ID_FROM_AIRPLAY_DEVICE" + " Airplay_Office_Staus")
  }
end
```

```js
rule"MAIN Volume"
when
  Item Volume_Main received update
then
  executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "volume " + Volume_Main.state + " Computer")
end
```

```js
rule"Volume Office"
when
  Item Volume_Office received update
then
  executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "volume "+ Volume_Office.state + " ID_FROM_AIRPLAY_DEVICE")
end
```

```js
rule"Play"
when
  Item iTunes_Play changed from OFF to ON
then
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "play")
end
```

```js
rule"Toogle Play/Pause"
when
  Item iTunes_PlayPause changed
then
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "playpause")
end
```

```js
rule"Pause"
when
  Item iTunes_Pause changed from OFF to ON
then
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "pause")
end
```

```js
rule"Stop"
when
  Item iTunes_Stop changed from OFF to ON
then
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "stop")
end
```

```js
rule"Previous"
when
  Item iTunes_Previous changed from OFF to ON
then
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "previous")
end
```

```js
rule"Next"
when
  Item iTunes_Next changed from OFF to ON
then
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "next")
end
```

```js
rule"Mute"
when
  Item iTunes_Mute received update
then
  if (iTunes_Mute.state == ON){
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "mute " + "true")
  }
  else{
    executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "mute " + "false")
  }
end
```

```js
rule"Shuffle"
when
  Item Shuffle_Item received update
then
  if (Shuffle_Item.state == 0) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "shuffle " + "off")
  if (Shuffle_Item.state == 1) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "shuffle " + "songs")
  if (Shuffle_Item.state == 2) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "shuffle " + "albums")
  if (Shuffle_Item.state == 3) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "shuffle " + "groupings")
end
```

```js
rule"Repeat"
when
  Item Repeat_Item received update
then
  if (Repeat_Item.state == 0) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "repeat "+ "off")
  if (Repeat_Item.state == 1) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "repeat "+ "one")
  if (Repeat_Item.state == 2) executeCommandLine("/etc/openhab2/scripts/oh_itunes-api.sh " + "repeat "+ "all")
end
```

```
  Playlisten Updaten -> sudo bash /etc/openhab2/scripts/oh_refresh_playlist_itunes-api.sh
```
