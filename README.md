# HalfHour Server

> You deserve privacy

A privacy-focused, ephemeral meeting server built with Deno. HalfHour enables secure, temporary one-on-one connections between meeting organizers and attendees with end-to-end encryption support.

## Features

- **🔒 Privacy-First**: End-to-end encryption support with public key exchange
- **⏱️ Ephemeral Meetings**: Time-bound sessions that automatically clean up
- **💬 Real-Time Communication**: WebSocket-based chat with typing indicators
- **📁 File Sharing**: Secure file uploads and downloads with storage limits
- **🎫 Tag-Based Access**: Meeting tags for organizers, response tags for attendees
- **🔐 Session Management**: JWT-based authentication with automatic expiration
- **📊 Statistics**: Built-in analytics for tracking usage
- **🧹 Auto-Cleanup**: Scheduled tasks to remove expired data
- **🚦 Rate Limiting**: Protection against abuse

## Tech Stack

- **Runtime**: [Deno](https://deno.land/) 2.x
- **Framework**: [Hono](https://hono.dev/) - Ultra-fast web framework
- **Database**: [SurrealDB](https://surrealdb.com/) - Multi-model database
- **Storage**: Backblaze B2 - Object storage for files
- **Validation**: Zod - TypeScript-first schema validation
- **Authentication**: JWT (via @panva/jose)
- **Scheduling**: Cron - Automated cleanup tasks

## Prerequisites

- Deno 2.x or higher
- SurrealDB instance
- Backblaze B2 account (for file storage)

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd halfhour
```

2. Create a `.env` file with the required environment variables (see [Configuration](#configuration))

3. Run the server:
```bash
# Development mode with hot reload
deno task dev

# Production mode
deno task start
```

## Configuration

Create a `.env` file in the root directory with the following variables:

### Server Configuration
```env
PORT=6002                          # Server port (default: 6002)
CLIENT_URL=http://localhost:3000   # Allowed CORS origin (your frontend URL)
```

### Database Configuration
```env
DB_URL=ws://localhost:8000         # SurrealDB connection URL
DB_USER=root                       # Database username
DB_PASS=root                       # Database password
DB_NS=halfhour                     # Database namespace
DB_DB=halfhour                     # Database name
```

### Storage Configuration (Backblaze B2)
```env
BB_BUCKET_ID=your-bucket-id        # Backblaze B2 bucket ID
BB_ENDPOINT_API=https://...        # B2 API endpoint
BB_KEY_ID=your-key-id              # B2 application key ID
BB_KEY=your-application-key        # B2 application key
```

## API Routes

### Organizer Routes (`/organiser`)
- `POST /organiser` - Create a new meeting
- `GET /organiser` - Get organizer details and meeting info
- `POST /organiser/auth/start` - Initiate organiser authentication
- `POST /organiser/auth/finish` - Finish organiser authentication

### Attendee Routes (`/attendee`)
- `POST /attendee` - Request to join a meeting
- `GET /attendee` - Get attendee details and meeting status
- `POST /attendee/auth/start` - Initiate attendee authentication
- `POST /attendee/auth/finish` - Finish attendee authentication

### Meeting Routes (`/meeting`)
- `POST /meeting/connect` - Approve attendee connection (organizer only)
- `GET /meeting/connection/:id` - Get meeting connection details
- `GET /meeting/realtime/:id` - WebSocket endpoint for real-time chat

### Session Routes (`/session`)
- `GET /session?id=<session_id>` - Validate a session
- `DELETE /session` - Logout/delete session

### Storage Routes (`/storage`)
- `GET /storage/connection/:id?file=<file_id>` - Download a file
- `POST /storage/connection/:id` - Upload a file

### Statistics Routes (`/statistics`)
- `GET /statistics/jobs` - Get cleanup tasks statuses
- `GET /statistics/meetings` - Get statistics related to meetings
- `GET /statistics/graphical` - Get data for plotting graphical statistics

## WebSocket Protocol

The real-time meeting endpoint (`/meeting/realtime/:id`) uses WebSocket with the following message types:

### Client → Server
```json
{
  "type": "text",
  "data": {
    "message": "Hello!",
    "file": "optional-file-id"
  }
}
```

```json
{
  "type": "typing"
}
```

```json
{
  "type": "joined"
}
```

```json
{
  "type": "left"
}
```

### Server → Client
```json
{
  "type": "text",
  "data": {
    "id": "message-id",
    "body": "Hello!",
    "has_attachment": false,
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

```json
{
  "type": "typing",
  "data": {
    "from": "user-id",
    "message": "Typing...",
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

## Docker Support

Build and run with Docker:

```bash
# Build the image
docker build -t halfhour-server .

# Run the container
docker run -p 6002:6002 --env-file .env halfhour-server
```

## Database Schema

The server automatically creates and manages the following tables:
- `organiser` - Meeting organizers
- `attendee` - Meeting attendees
- `session` - Authentication sessions
- `login` - Login records
- `connects_with` - Active meeting connections (relation table)
- `requests_to` - Pending connection requests (relation table)
- `conversation_with` - Chat messages (relation table)
- `file` - File metadata
- `opened_by` - File access tracking (relation table)

## Cleanup Tasks

The server runs scheduled cleanup tasks to remove expired data:
- **clean_attendee**: Removes expired attendee records
- **clean_organiser**: Removes expired organizer/meeting records
- **clean_session**: Removes expired session tokens
- **clean_file**: Removes orphaned or expired files

## Security Features

- **CORS Protection**: Configurable allowed origins
- **Rate Limiting**: Built-in rate limiter on sensitive endpoints
- **Session Expiration**: Time-limited authentication tokens
- **Password Hashing**: Client-side password hashing with salts
- **Timing-Safe Comparison**: Protection against timing attacks
- **End-to-End Encryption**: Public key infrastructure for secure communications

## Development

```bash
# Run with hot reload
deno task dev

# Format code
deno fmt

# Lint code
deno lint
```

## License

This project is licensed under the GNU Affero General Public License v3.0 (AGPL-3.0).

See the [LICENSE](LICENSE) file for details.

## Architecture Overview

```
halfhour-server/
├── database/           # Database connection and schema management
│   ├── config.ts      # SurrealDB connection
│   ├── startup.ts     # Database initialization
│   └── kv.ts          # Key-value store utilities
├── routes/            # API route handlers
│   ├── attendee/      # Attendee management
│   ├── organiser/     # Meeting organizer management
│   ├── meeting/       # Real-time meeting functionality
│   ├── session/       # Authentication sessions
│   ├── storage/       # File upload/download
│   └── statistics/    # Usage analytics
├── misc/              # Utility functions and tasks
│   ├── clean_*.task.ts    # Scheduled cleanup jobs
│   ├── verify_request.util.ts  # Authentication middleware
│   ├── rate_limiter.util.ts    # Rate limiting
│   └── *.util.ts      # Various utilities
└── main.ts            # Application entry point
```

## Contributing

Contributions are welcome! Please ensure your code:
- Follows the existing code style
- Passes linting (`deno lint`)
- Is properly formatted (`deno fmt`)
- Includes appropriate error handling

---

**Note**: This server is designed to work with a compatible frontend client. Make sure to set the `CLIENT_URL` environment variable to your frontend's URL for proper CORS configuration.