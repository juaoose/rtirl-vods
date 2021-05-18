https://rtirl.com/auth/twitch/callback?code=4ezqwrtg3ictyjcyx1m9fgd6d4ps8n&scope=user%3Aread%3Aemail&state=44aaf47c86b0b2c85906f824700553e821ba01ce


https://id.twitch.tv/oauth2/authorize?response_type=code&client_id=a6rvm91td9m00vach7ozphvbm80jkv&redirect_uri=https%3A%2F%2Frtirl.com%2Fauth%2Ftwitch%2Fcallback&scope=user%3Aread%3Aemail&state=86f919598079dbb6ab4a4a2bf463be95ee4ad5b8

https://rtirl.com/auth/twitch/callback?code=fw9ea8r2qgce04lj9w9qg8et9ce7vc&scope=user%3Aread%3Aemail&state=86f919598079dbb6ab4a4a2bf463be95ee4ad5b8

curl -X GET "https://api.twitch.tv/helix/videos?id=1023837339" -H "Authorization: Bearer 9r3eh8rl85ghlv0fu9su7ez4xbqfq2" -H "Client-Id: nddyowznjhwjvewpt9r7dd4wwrprz0"


https://id.twitch.tv/oauth2/authorize?client_id=nddyowznjhwjvewpt9r7dd4wwrprz0&redirect_uri=http://localhost&response_type=token&scope=channel:manage:videos

https://dev.twitch.tv/docs/api/reference#get-videos

curl -X GET "https://api.twitch.tv/helix/videos?id=1023837339" -H "Authorization: Bearer 9r3eh8rl85ghlv0fu9su7ez4xbqfq2" -H "Client-Id: nddyowznjhwjvewpt9r7dd4wwrprz0"
{"data":[{"id":"1023837339","stream_id":"42079406220","user_id":"158394109","user_login":"muxfd","user_name":"muxfd","title":"nyc biking - ☀️ inwood hill park","description":"","created_at":"2021-05-15T19:06:13Z","published_at":"2021-05-15T19:06:13Z","url":"https://www.twitch.tv/videos/1023837339","thumbnail_url":"https://static-cdn.jtvnw.net/cf_vods/dgeft87wbj63p/42942302ff21c251487b_muxfd_42079406220_1621105563//thumb/thumb0-%{width}x%{height}.jpg","viewable":"public","view_count":2276,"language":"en","type":"archive","duration":"4h27m50s","muted_segments":null}],"pagination":{}}

Wouldnt we likely fail at matching times between VOD and GPX/stored data by using 


todays stream at like 3:24pm

https://codepen.io/OttyLab/pen/GRjBjwq
https://docs.mapbox.com/mapbox-gl-js/example/animate-a-line/


Hello again!

I made progress with gpx loading and an initial animation version for that track, I'm working on making the animation better match the video speed but I have some doubts:
- From the track your shared I see that most of the time you perform a read every second, but could there be a scenario where gps data is missing in the session? Maybe if the phone/app crashes you could have records like this t1 `2021-05-15T19:09:32.000Z` and t2 `2021-05-15T19:19:32.000Z` I believe this could be important so that the animation doesn't just starts going faster than the VOD.
- I believe there are cases (I guess most of the times) where the VOD and the track won't match in length (starting stream or app first) so I think I'll add a way to modify the offset to make both match. Initially I was looking into the API to see if there was something like `occured_at` but didn't really find it. (`https://api.twitch.tv/helix/videos?id=1023837339` looked promising but I still have to check if `created_at` can help match both video and gpx tracks)
- I started playing with it using Vue but I noticed you use React for RTIRL, should I swap right away?