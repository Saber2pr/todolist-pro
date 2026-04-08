### Image Information

- Image Name: `saber2pr/todolist-app:master`

### Method 1: Direct Pull Deployment

```bash
docker pull saber2pr/todolist-app:master
docker run -d -p 3000:3000 saber2pr/todolist-app:master
```

### Method 2: Deployment via Dockerfile

Create a `Dockerfile`:

```dockerfile
FROM saber2pr/todolist-app:master
EXPOSE 3000
```

### Access

After starting, access http://localhost:3000/

### Webhook Data Synchronization

Once deployed, you can use a Webhook to automatically sync data to the web after editing Todo.

1. Open the application's **Setting** page
2. Fill in the Webhook address bar: `http://localhost:3000/api` (i.e., deployment address + `/api`)
3. After saving, when you edit Todo, the data will automatically sync to the web side

> If the deployment address is not localhost:3000, replace the Webhook address with the corresponding `http://<your-host>:<port>/api`.