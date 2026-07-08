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

## Bug Reproduction

**Issue #2:**

I reproduced this bug by adding a friend to a user, and having that friend play a song the previous day near midnight, than ran the command the next day. This caused the friend to show up as playing that song, even though they weren't playing it that day, but the previous day.

**Issue #3:**

I reproduced this bug by following exactly what Simone did. I ran a song search for "anthem" and found only 1 result, but after some more digging, I found that the multiple entries returned were being collapsed into one, so once I temporarily stopped this behavior, and got returned three different results from the "anthem" search.

**Issue #5**

Reproducing this issue was very simplistic I tested all three playlists created in seed_data, and when launcing all three song lists using `/playlists/<playlist_id>/songs` without exemption it did not show the last song the playlist should've included.