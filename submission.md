# Project submission

---

## Codebase Map
The first thing I looked at in this file was models.py, which defines 7 SQLAlchemy models: 
   
    1) User
    2) Tag
    3) Song
    4) ListeningEvent
    5) Rating
    6) Playlist
    7) Notification

Each one of these stores their namesake, using necessary information. Tag, mainly being used for the purpose of song, rather than it's own real storage.

**Data flow-**
user reads a notification: 

`POST/users/notifications/<notification_id>/read` in `routes/users.py` calls `notification_service.mark_as_read()`. The function called intakes an `notification_id` for the notification the user is marking as read. The function then edits the `notification.read` status of that notification to true and commits the change. It also ensures that the notification ID that was actually provided is real, and returns an error saying that the `notification_id` was not found.

**Noticed patterns:**
One pattern that was very obvious when looking at the program was the structure of each of the routes was nearly identical. Their structure is running a program from one of the services, and throwing an error when an input causes an issue. All the logic is done outside of the actual routes, which overall makes sense.


**Issue #5**

Reproducing this issue was very simplistic I tested all three playlists created in seed_data, and when launcing all three song lists using `/playlists/<playlist_id>/songs` without exemption it did not show the last song the playlist should've included.

## Root cause Analysis #1

**Issue #2- Friends Listening Now shows people from yesterday**

**Reproduction:**

I reproduced this bug by adding a friend to a user, and having that friend play a song the previous day near midnight, than ran the command the next day. This caused the friend to show up as playing that song, even though they weren't playing it that day, but the previous day.

**How I found the root cause:**

To find the root issue, I started in `routes/feed` looking at the problimatic class, `listening_now`. From there I looked at what it called, which was `get_friends_listening_now`, found where it was located via the imports, navigated to that file in the repo, then Ctrl+f in the file to find the class. I was confident I was in the right place once I looked at the time checking logic used in that class, which would cause the exact issues reported in the bug report, and would explain my independent reproduction.

**The root cause:**

The issue present in this class was that in order to figure out whether the last song was played today, the program checked if it was played in the last 24 hours rather than being day dependent. The fact that the calculation was done by time rather than by day caused the issue as at 9am, the song from 11pm the day previously would not have expired for the day, as 24 hours hadened passed, and no new song was played after that time, leading to the bug.

**My fix and side-effect check:**

I fixed the issue by changing the cutoff from `datetime.now(timezone.utc) - RECENT_THRESHOLD (24 hours)` to `datetime.now(timezone.utc).replace(hour=0, minute=0, second=0, microsecond=0)` This changed the logic for the cutoff to always be at midnight rather than 24 hours after the song is played, which fixes the root issue. There wasn't much side-effect checks that I had to do for this change as the change affects nothing besides this class, which is only called for that function, so once I checked that `listening_now` worked properly, I completed the bug fix.

## Root Cause Analysis #2

**Issue #3- The same song keeps showing up twice in search**

**Reproduction:**

I reproduced this bug by following exactly what Simone did. I ran a song search for "anthem" and found only 1 result, but after some more digging, I found that the multiple entries returned were being collapsed into one, so once I temporarily stopped this behavior, and got returned three different results from the "anthem" search.

**How I found the root cause:**

To find the root issue, I started in `routes/songs` looking at the problimatic class, `search`. From there I looked at what it called, which was `search_song`, found where it was located via the imports, navigated to that file in the repo, then Ctrl+f in the file to find the class. I was initially unconfident I had found the correct issue in the class, as I was unfamiliar with what the class was doing, so I turned to AI. I gave the AI the class, the bug report, and some information from my own testing. It then pointed me to what could have been the issue, and after some further experimentation with the bug, the diagnosis it provided me appeared to be correct, and the issue.

**The root cause:**

The primary issue that caused this bug was the `.outerjoin(song_tags, song.id == song_tags.c.song_id)` command in the class. This caused each song to show up equal to the amount of tags the song had, so the song Simone searched for appeared three times, songs with one tag appeared once, and songs with zero tags appeared no times. 

**My fix and side-effect check**

My primary fix for this issue was just entirely removing the problematic line as it was entirely unnecessary, as the information it supposidly gathers is just added later in the class. As with the previous issue, the only assurance test I had to make was to make sure that `search` in `routes/songs` still works properly, as `search_song` is used nowhere else in the program. Once I completed these tests, the issue was officially fixed.

