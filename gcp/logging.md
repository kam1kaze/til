Work with logs in a console

```
# Copy a neccessary path
gcloud logging logs list

# Output last 30 lines from the $PATH, reverse the output order with `tac`
gcloud logging read $PATH --limit=30 --format='value(textPayload)' | tac
```
