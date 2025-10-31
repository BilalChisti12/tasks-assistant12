# Todo App with Email Deadline Reminders - Quick Start Guide

## Overview

This is a complete Todo application with automated email deadline reminders. The system consists of:
- **Frontend**: Responsive single-page application (see demo)
- **Backend**: Node.js/Express REST API
- **Database**: PostgreSQL with Prisma ORM
- **Job Queue**: Bull with Redis for reliable email scheduling
- **Email**: Nodemailer with configurable SMTP

## Prerequisites

- Node.js 18+ and npm
- PostgreSQL 14+
- Redis 7+
- SMTP credentials (Gmail, SendGrid, etc.)
- Git

## Quick Setup (Development)

### 1. Clone and Install

```bash
git clone <your-repo-url>
cd todo-backend
npm install
```

### 2. Configure Environment

Create `.env` file:

```env
# Database
DATABASE_URL="postgresql://postgres:password@localhost:5432/todo?schema=public"

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# Email (Gmail example)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
EMAIL_FROM=your-email@gmail.com

# Server
PORT=3000
NODE_ENV=development
CORS_ORIGIN=http://localhost:8080
```

### 3. Setup Database

```bash
# Install PostgreSQL (macOS)
brew install postgresql@14
brew services start postgresql@14

# Create database
createdb todo

# Run migrations
npx prisma migrate dev --name init
npx prisma generate
```

### 4. Setup Redis

```bash
# Install Redis (macOS)
brew install redis
brew services start redis

# Verify Redis is running
redis-cli ping  # Should return PONG
```

### 5. Start Application

**Terminal 1 - API Server:**
```bash
npm run dev
```

**Terminal 2 - Email Worker:**
```bash
npm run dev:worker
```

**Terminal 3 - Frontend (optional):**
```bash
# Serve the frontend HTML file
python3 -m http.server 8080
# Or use Live Server in VS Code
```

### 6. Test the Application

```bash
# Create a test task
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test email reminder",
    "description": "This should send an email",
    "email": "your-email@example.com",
    "deadline": "2025-11-01T14:00:00Z",
    "priority": "HIGH"
  }'

# Get all tasks
curl http://localhost:3000/api/tasks
```

## Production Deployment Options

### Option 1: Heroku (Easiest)

1. **Install Heroku CLI**
   ```bash
   brew install heroku/brew/heroku
   ```

2. **Create Heroku App**
   ```bash
   heroku login
   heroku create your-app-name
   ```

3. **Add Add-ons**
   ```bash
   heroku addons:create heroku-postgresql:mini
   heroku addons:create heroku-redis:mini
   ```

4. **Set Environment Variables**
   ```bash
   heroku config:set NODE_ENV=production
   heroku config:set SMTP_HOST=smtp.sendgrid.net
   heroku config:set SMTP_USER=apikey
   heroku config:set SMTP_PASS=SG.your-api-key
   heroku config:set EMAIL_FROM=noreply@yourdomain.com
   ```

5. **Create Procfile**
   ```
   web: node server.js
   worker: node workers/emailWorker.js
   ```

6. **Deploy**
   ```bash
   git add .
   git commit -m "Initial deployment"
   git push heroku main
   
   # Run migrations
   heroku run npx prisma migrate deploy
   
   # Scale worker
   heroku ps:scale worker=1
   
   # View logs
   heroku logs --tail
   ```

7. **Access Your App**
   ```
   https://your-app-name.herokuapp.com
   ```

### Option 2: Render (Modern Alternative)

1. **Go to render.com and sign up**

2. **Create PostgreSQL Database**
   - New → PostgreSQL
   - Name: `todo-db`
   - Copy the Internal Database URL

3. **Create Redis Instance**
   - New → Redis
   - Name: `todo-redis`
   - Copy the Redis URL

4. **Create Web Service**
   - New → Web Service
   - Connect your GitHub repo
   - Name: `todo-api`
   - Build Command: `npm install && npx prisma generate && npx prisma migrate deploy`
   - Start Command: `node server.js`
   - Add Environment Variables:
     - DATABASE_URL: (from step 2)
     - REDIS_HOST: (parse from Redis URL)
     - REDIS_PORT: (parse from Redis URL)
     - SMTP_HOST, SMTP_USER, SMTP_PASS, EMAIL_FROM

5. **Create Background Worker**
   - New → Background Worker
   - Same repo as web service
   - Start Command: `node workers/emailWorker.js`
   - Same environment variables

6. **Deploy**
   - Render auto-deploys on git push

### Option 3: Railway

1. **Install Railway CLI**
   ```bash
   npm i -g @railway/cli
   ```

2. **Login and Initialize**
   ```bash
   railway login
   railway init
   ```

3. **Add Services**
   ```bash
   railway add --plugin postgresql
   railway add --plugin redis
   ```

4. **Deploy**
   ```bash
   railway up
   ```

5. **Configure via Dashboard**
   - Go to railway.app
   - Add environment variables
   - Add worker service with `node workers/emailWorker.js`

### Option 4: Docker Compose (Self-hosted)

1. **Create docker-compose.yml**
   ```yaml
   version: '3.8'
   
   services:
     app:
       build: .
       ports:
         - "3000:3000"
       environment:
         DATABASE_URL: postgresql://postgres:password@db:5432/todo
         REDIS_HOST: redis
         SMTP_HOST: ${SMTP_HOST}
         SMTP_USER: ${SMTP_USER}
         SMTP_PASS: ${SMTP_PASS}
       depends_on:
         - db
         - redis
     
     worker:
       build: .
       command: node workers/emailWorker.js
       environment:
         DATABASE_URL: postgresql://postgres:password@db:5432/todo
         REDIS_HOST: redis
         SMTP_HOST: ${SMTP_HOST}
         SMTP_USER: ${SMTP_USER}
         SMTP_PASS: ${SMTP_PASS}
       depends_on:
         - db
         - redis
     
     db:
       image: postgres:14-alpine
       environment:
         POSTGRES_PASSWORD: password
         POSTGRES_DB: todo
       volumes:
         - pgdata:/var/lib/postgresql/data
     
     redis:
       image: redis:7-alpine
       volumes:
         - redisdata:/data
   
   volumes:
     pgdata:
     redisdata:
   ```

2. **Deploy**
   ```bash
   docker-compose up -d
   ```

## Email Provider Setup

### Gmail Setup

1. Enable 2-factor authentication on your Google account
2. Generate App Password:
   - Google Account → Security → 2-Step Verification → App passwords
   - Select "Mail" and "Other (Custom name)"
   - Copy the 16-character password

3. Use these settings:
   ```env
   SMTP_HOST=smtp.gmail.com
   SMTP_PORT=587
   SMTP_USER=your-email@gmail.com
   SMTP_PASS=your-16-char-app-password
   ```

### SendGrid Setup

1. Sign up at sendgrid.com
2. Create API key:
   - Settings → API Keys → Create API Key
   - Full Access permissions
   - Copy the key (starts with SG.)

3. Use these settings:
   ```env
   SMTP_HOST=smtp.sendgrid.net
   SMTP_PORT=587
   SMTP_USER=apikey
   SMTP_PASS=SG.your-api-key
   ```

### Mailgun Setup

1. Sign up at mailgun.com
2. Get SMTP credentials from dashboard

3. Use these settings:
   ```env
   SMTP_HOST=smtp.mailgun.org
   SMTP_PORT=587
   SMTP_USER=postmaster@your-domain.mailgun.org
   SMTP_PASS=your-password
   ```

## API Endpoints

### Tasks

- `POST /api/tasks` - Create task
- `GET /api/tasks` - List all tasks
- `GET /api/tasks?status=active` - Filter by status
- `GET /api/tasks?email=user@example.com` - Filter by email
- `GET /api/tasks/:id` - Get single task
- `PATCH /api/tasks/:id` - Update task
- `DELETE /api/tasks/:id` - Delete task

### Health Check

- `GET /health` - Server health status

### Example Request

```bash
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Finish quarterly report",
    "description": "Complete Q4 analysis and submit to management",
    "email": "manager@company.com",
    "deadline": "2025-12-15T17:00:00+05:30",
    "priority": "HIGH",
    "timezone": "Asia/Kolkata"
  }'
```

## Troubleshooting

### Database Connection Issues

```bash
# Check PostgreSQL is running
pg_isready

# Check connection
psql postgresql://postgres:password@localhost:5432/todo

# Reset database
npx prisma migrate reset
```

### Redis Connection Issues

```bash
# Check Redis is running
redis-cli ping

# Check connection
redis-cli
> KEYS *
> QUIT
```

### Email Not Sending

1. Check SMTP credentials in `.env`
2. Check worker logs: `heroku logs --tail -p worker`
3. Test email manually:
   ```javascript
   const nodemailer = require('nodemailer');
   
   const transporter = nodemailer.createTransport({
     host: 'smtp.gmail.com',
     port: 587,
     auth: {
       user: 'your-email@gmail.com',
       pass: 'your-app-password'
     }
   });
   
   transporter.sendMail({
     from: 'your-email@gmail.com',
     to: 'test@example.com',
     subject: 'Test',
     text: 'Test email'
   }).then(console.log).catch(console.error);
   ```

### Worker Not Processing Jobs

```bash
# Check Redis has jobs
redis-cli
> KEYS bull:email-reminders:*
> LLEN bull:email-reminders:wait

# Restart worker
heroku restart worker

# Check worker logs
heroku logs --tail --ps worker
```

## Monitoring

### View Logs

```bash
# Heroku
heroku logs --tail

# Railway
railway logs

# Docker
docker-compose logs -f
```

### Bull Board (Job Dashboard)

Add to server.js:

```javascript
const { createBullBoard } = require('@bull-board/api');
const { BullAdapter } = require('@bull-board/api/bullAdapter');
const { ExpressAdapter } = require('@bull-board/express');

const serverAdapter = new ExpressAdapter();
createBullBoard({
  queues: [new BullAdapter(emailQueue)],
  serverAdapter
});

app.use('/admin/queues', serverAdapter.getRouter());
```

Access at: `http://localhost:3000/admin/queues`

## Security Checklist

- [ ] Environment variables secured (not in git)
- [ ] CORS configured for production domain
- [ ] Rate limiting enabled
- [ ] Helmet security headers configured
- [ ] Input validation on all endpoints
- [ ] HTTPS enabled in production
- [ ] Database credentials rotated regularly
- [ ] API keys stored in secrets manager
- [ ] Logs sanitized (no sensitive data)
- [ ] Error messages don't expose system details

## Performance Tips

1. **Database Indexing**: Already configured in Prisma schema
2. **Connection Pooling**: Prisma handles automatically
3. **Redis Persistence**: Configure AOF for durability
4. **Email Rate Limiting**: Respect provider limits
5. **Worker Scaling**: Scale based on job queue length

## Testing

### Manual Testing Checklist

- [ ] Create task via API
- [ ] Task appears in frontend
- [ ] Edit task updates deadline
- [ ] Delete task removes from database
- [ ] Email sent at deadline time
- [ ] Completed tasks don't send email
- [ ] Overdue tasks handled correctly
- [ ] Timezone conversion works
- [ ] Error handling for invalid input
- [ ] Rate limiting works

### Automated Testing

```bash
npm test
```

## Next Steps

1. **Add Authentication**
   - Implement JWT or OAuth
   - User registration/login
   - Associate tasks with users

2. **Add Features**
   - Recurring tasks
   - Task templates
   - File attachments
   - Push notifications
   - Calendar export (ICS)

3. **Improve Reliability**
   - Add Sentry for error tracking
   - Set up uptime monitoring
   - Implement circuit breakers
   - Add database backups

4. **Scale**
   - Add Redis Cluster
   - Implement database read replicas
   - Add CDN for frontend
   - Implement caching layer

## Resources

- **Express.js**: https://expressjs.com/
- **Prisma**: https://www.prisma.io/docs
- **Bull**: https://github.com/OptimalBits/bull
- **Nodemailer**: https://nodemailer.com/
- **Heroku Dev Center**: https://devcenter.heroku.com/
- **Render Docs**: https://render.com/docs

## Support

For issues:
1. Check logs first
2. Review environment variables
3. Test components individually
4. Check GitHub issues for similar problems

## License

MIT License - feel free to use for personal or commercial projects.

---

**Last Updated**: October 31, 2025
