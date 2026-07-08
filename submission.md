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