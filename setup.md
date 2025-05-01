
## Project Setup and Commands:

1. Fork this repository to your own account.

2. Clone the project repository to your local machine.

3. Create a local .env file with your [MailTrap](https://mailtrap.io/) SMTP settings. Mailtrap allows you to view emails when you test the site manually. When running pytest, the system uses a Mock to simulate sending emails but doesn't actually send them. You also have to add the the variables for MinIO if you implement the same feature , the variables like endpoint, accesskey ,secretkey and bucketname.

4. Alembic and Pytest:
  - When you run Pytest, it deletes the user table but doesn't remove the Alembic table. This can cause Alembic to get out of sync.
  - To resolve this, drop the Alembic table and run the migration (`docker compose exec fastapi alembic upgrade head`) when you want to manually test the site through `http://localhost/docs`.
  - If you change the database schema, delete the Alembic migration, the Alembic table, and the users table. Then, regenerate the migration using the command: `docker compose exec fastapi alembic revision --autogenerate -m 'initial migration'`.
  - Since there is no real user data currently, you don't need to worry about database upgrades, but Alembic is still required to install the database tables.

5. Run the project:
  - `docker compose up --build`
  - Set up PGAdmin at `localhost:5050` (see docker compose for login details)
  - View logs for the app: `docker compose logs fastapi -f`
  - Run tests: `docker compose exec fastapi pytest`

6. Set up the project with DockerHub deployment as in previous assignments for email testing. Enable issues in settings, create the production environment, and configure your DockerHub username and token. You don't need to add MailTrap, but if you want to, you can add the values to the production environment's variables.
 
 # User Profile Picture Upload with MinIO Integration

## How MinIO Works

MinIO is a high-performance object storage system that provides:
1. S3-compatible API for cloud-native applications
2. Distributed architecture for scalability
3. Object storage rather than traditional filesystem

## Upload Process

When a user uploads a profile picture, the following steps occur:

1. **User Initiates Upload**
   - User selects image file through interface
   - Frontend validates basic format and size

2. **Server-Side Processing**
   - API endpoint receives multipart form data
   - System authenticates and authorizes the user
   - Application validates file type, size, and content

3. **MinIO Storage Operations**
   - Application generates unique filename with UUID
   - System connects to MinIO using configured credentials
   - File is uploaded to designated bucket in MinIO
   - MinIO returns confirmation of successful storage

4. **Database Integration**
   - User profile record is updated with reference to image
   - Success response returned to user interface

5. **Image Access Flow**
   - When profile is viewed, system generates presigned URL
   - Presigned URL provides temporary authenticated access
   - Image loads directly from MinIO to user interface

## Security Considerations

The implementation includes several security measures:

1. **Authentication & Authorization**
   - Only authenticated users can upload images
   - Users can only modify their own profiles (unless admin)

2. **File Validation**
   - Strict validation of file types (JPEG, PNG, GIF only)
   - Size limitation (max 5MB) to prevent DoS attacks
   - Content-type verification beyond extension checking

3. **Secure Storage**
   - Files stored with random UUIDs to prevent guessing
   - Proper bucket policies to prevent unauthorized access
   - Optional encryption at rest for sensitive deployments

4. **Secure Access**
   - Presigned URLs with short expiration times
   - No direct public access to storage buckets
   - HTTPS for all communications with MinIO

## Implementation Details

### Configuration

MinIO is configured with such parameters:

```yaml
MINIO_ENDPOINT: "sample:8080"
MINIO_ACCESS_KEY: "samplead"
MINIO_SECRET_KEY: "dummypassword"
MINIO_BUCKET_NAME: "profile-pictures"
```

### API Endpoint

```
POST /users/{user_id}/profile-picture
```