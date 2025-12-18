# User Management — MongoDB and MinIO

This guide covers creating users and managing access for MongoDB and MinIO in your Altar stack.

---

## 1. MongoDB

### Create a new user

Connect to MongoDB:
```bash
docker exec -it mongo mongosh -u admin -p your_password --authenticationDatabase admin
```

Inside mongosh, create a user with read-write access:
```javascript
use sacred
db.createUser({
  user: "username",
  pwd:  "password123",
  roles: [ { role: "readWrite", db: "sacred" } ]
})
```

### Available roles

| Role        | Description                          |
|-------------|--------------------------------------|
| `read`      | Read-only access to the database     |
| `readWrite` | Read and write access to the database|

### Example: Read-only user
```javascript
use sacred
db.createUser({
  user: "readonly_user",
  pwd:  "password123",
  roles: [ { role: "read", db: "sacred" } ]
})
```

---

## 2. MinIO

The easiest way to manage MinIO users is through the web console.

### Access the console

1. Open http://localhost:9001 (or your configured console port)
2. Log in with your MinIO admin credentials (`MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`)

### Create a policy

1. Go to **Administrator** → **Policies**
2. Click **Create Policy +** in the top right
3. Enter a policy name and paste the JSON

#### Read/Write policy example

Allows read and write access to specific buckets:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowBucketReadWrite",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:HeadBucket",
                "s3:ListAllMyBuckets",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"
            ]
        }
    ]
}
```

> **Note:** Users can read/write to existing buckets but cannot create new ones. Create buckets as admin: **Administrator** → **Buckets** → **Create Bucket**.

> **Important:** Bucket names can only contain lowercase letters (`a-z`), numbers (`0-9`), and hyphens (`-`).

#### Read-only policy example

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowReadOnly",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:HeadBucket",
                "s3:ListAllMyBuckets",
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"
            ]
        }
    ]
}
```

### Create a user

1. Go to **Administrator** → **Identity** → **Users**
2. Click **Create User +**
3. Enter a username and password (these are console login credentials)
4. Assign the policies you want

### Create access keys

To use MinIO with S3-compatible tools (like AltarSender), users need Access Keys:

**Option 1:** User creates their own
- User logs into the console with their credentials
- Creates an Access Key / Secret Key pair
- Adds them to their environment variables

**Option 2:** Admin creates for user
- Admin creates the key pair in the console
- Transmits the credentials securely to the user

Access Keys inherit the permissions of the user they belong to.
