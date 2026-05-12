# Issue #93: Real-Time Features - Implementation Guide

## Overview

This guide documents the implementation of real-time features for NexaSphere, including:
- **WebSocket Communication** (Socket.IO) for bidirectional updates
- **Push Notifications** (Firebase Cloud Messaging) for mobile/web
- **Transactional Emails** (SendGrid) for confirmations and reminders
- **Server-Sent Events (SSE)** for admin dashboard streaming
- **Service Worker** for PWA offline support

## Acceptance Criteria Status

✅ **WebSocket bidirectional communication via Socket.IO**
- Socket.IO configuration at `server/config/socket.js`
- Real-time event broadcasting to users and admin rooms
- Automatic reconnection with exponential backoff

✅ **Push notifications for registration, promotions, reminders**
- Firebase Cloud Messaging integration at `server/services/pushNotificationService.js`
- Frontend push subscription at `src/utils/pushNotificationClient.js`
- Service Worker push handling at `public/sw.js`

✅ **Transactional emails via SendGrid**
- Email service at `server/services/emailService.js`
- Email templates for: registration, waitlist promotion, event reminders, attendance
- Automatic email sending on events

✅ **Event emitters for key business events**
- Event emitter at `server/services/eventEmitterService.js`
- Events: registration-confirmed, waitlist-promotion, event-reminder, attendance-marked
- Integrated with Socket.IO, push notifications, and email service

✅ **Server-Sent Events (SSE) for admin real-time updates**
- SSE service at `server/services/sseService.js`
- Admin stream route at `server/routes/adminStream.js`
- Live metrics and activity broadcasting

✅ **Service Worker for PWA support**
- Service Worker at `public/sw.js`
- PWA manifest at `public/manifest.json`
- Offline caching and background sync

✅ **Frontend integration**
- Socket.IO client at `src/utils/socketClient.js`
- Push notification client at `src/utils/pushNotificationClient.js`
- Example integration components

## Files Created/Modified

### Backend Real-Time Infrastructure

1. **server/config/socket.js** (NEW)
   - Socket.IO server configuration
   - Connection handling, user identification, rooms management
   - Broadcasting utilities for different scenarios

2. **server/services/pushNotificationService.js** (NEW)
   - Firebase Cloud Messaging initialization
   - Send to individual users, multicast, or topics
   - Topic subscription management

3. **server/services/emailService.js** (UPDATED)
   - Added registration confirmation emails
   - Waitlist promotion emails
   - Event reminder emails
   - Attendance confirmation emails
   - Bulk email capabilities

4. **server/services/eventEmitterService.js** (NEW)
   - Real-time event manager
   - Integrated Socket.IO, push notifications, email service
   - Event types: registration, promotion, reminder, attendance

5. **server/services/sseService.js** (NEW)
   - Server-Sent Events configuration
   - Client management and broadcasting
   - Heartbeat to keep connections alive

6. **server/routes/adminStream.js** (NEW)
   - SSE streaming endpoint for admin dashboard
   - Connected clients monitoring

### Frontend Real-Time Integration

7. **src/utils/socketClient.js** (NEW)
   - Socket.IO client wrapper
   - User identification and room management
   - Event listener registration
   - Connection status monitoring

8. **src/utils/pushNotificationClient.js** (NEW)
   - Service worker registration
   - Notification permission handling
   - Push subscription management
   - Local notification display

### PWA Configuration

9. **public/sw.js** (NEW)
   - Service worker for PWA support
   - Cache-first strategy for offline
   - Push notification handling
   - Background sync

10. **public/manifest.json** (UPDATED)
    - PWA metadata
    - App icons and screenshots
    - Shortcuts and share target

### Documentation & Examples

11. **BACKEND_INTEGRATION_EXAMPLE_ISSUE93.js** (NEW)
    - Backend setup and configuration
    - Event emission examples
    - Database integration points
    - Environment variables

12. **FRONTEND_INTEGRATION_EXAMPLE_ISSUE93.jsx** (NEW)
    - React component examples
    - Real-time hooks
    - Admin dashboard components
    - Connection status indicator

13. **.env.example.issue93** (NEW)
    - Complete environment variables template
    - Firebase, SendGrid, VAPID configuration
    - Development and production examples

14. **REAL_TIME_FEATURES_GUIDE.md** (THIS FILE)
    - Implementation overview
    - Setup instructions
    - API documentation

## Architecture

### Event Flow

```
User Action (register, mark attendance)
    ↓
API Endpoint (Express.js)
    ↓
Database Update
    ↓
Event Emitter (eventEmitterService)
    ↓
┌─────────────────────────────────────┐
│ 1. Socket.IO Broadcast              │ → Connected users
│ 2. Firebase Push Notification       │ → Mobile/web push
│ 3. SendGrid Email                   │ → Transactional email
│ 4. SSE Admin Stream                 │ → Admin dashboard
└─────────────────────────────────────┘
```

### Real-Time Communication Methods

**Socket.IO (WebSocket)**
- Bidirectional communication
- Used for: Live updates, admin notifications, user-to-user events
- Rooms: admin-room, notifications-room, events-room

**Push Notifications (FCM)**
- One-way notifications to devices
- Used for: Event reminders, waitlist promotions, registration confirmations
- Requires device subscription

**Server-Sent Events (SSE)**
- One-way server-to-client streaming
- Used for: Admin dashboard metrics, real-time activity feed
- Simple HTTP connection, automatic reconnection

**Email (SendGrid)**
- Transactional emails
- Used for: Confirmations, reminders, receipts
- Async sending, retry on failure

## Setup Instructions

### 1. Environment Variables

Create `.env` file with:
```bash
# Socket.IO
FRONTEND_URL=http://localhost:5173

# Firebase
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_SERVICE_ACCOUNT={"type":"service_account"...}
VAPID_PUBLIC_KEY=your-public-key
VAPID_PRIVATE_KEY=your-private-key

# SendGrid
SENDGRID_API_KEY=SG.xxxxx
SENDGRID_FROM_EMAIL=noreply@nexasphere.com
```

### 2. Backend Setup

```javascript
import express from 'express';
import { createServer } from 'http';
import { initializeSocketIO } from './config/socket.js';
import { initializeFirebase } from './services/pushNotificationService.js';
import { initializeSendGrid } from './services/emailService.js';

const app = express();
const httpServer = createServer(app);

initializeSocketIO(httpServer);
initializeFirebase();
initializeSendGrid();

httpServer.listen(3000);
```

### 3. Frontend Setup

```jsx
import { initializeSocket } from '@/utils/socketClient';
import { initializePushNotifications } from '@/utils/pushNotificationClient';

useEffect(() => {
  initializeSocket();
  initializePushNotifications(process.env.REACT_APP_VAPID_PUBLIC_KEY);
}, []);
```

### 4. Firebase Setup

1. Create Firebase project: https://console.firebase.google.com
2. Enable Cloud Messaging
3. Generate VAPID keys: `web-push generate-vapid-keys`
4. Download service account JSON
5. Add to environment variables

### 5. SendGrid Setup

1. Create SendGrid account: https://sendgrid.com
2. Create API key with Mail Send permission
3. Verify sender email
4. Add to SENDGRID_API_KEY

## API Endpoints

### Push Notifications

**Subscribe to push notifications**
```
POST /api/notifications/subscribe
Body: { subscription: PushSubscription }
```

**Unsubscribe from push notifications**
```
POST /api/notifications/unsubscribe
Body: { subscription: PushSubscription }
```

### Admin Stream

**Connect to real-time admin stream**
```
GET /api/admin/stream
Response: Server-Sent Events stream
```

**Get connected clients count**
```
GET /api/admin/stream/info
Response: { success: true, connectedClients: 5, timestamp: "..." }
```

### Monitoring (from Issue #92)

**Get real-time metrics**
```
GET /api/monitoring/metrics
Response: { avg_response_time, error_rate, request_count }
```

**Get error statistics**
```
GET /api/monitoring/errors/stats
Response: { total_errors, errors_by_status, top_endpoints }
```

## Socket.IO Events

### Client → Server

```javascript
socket.emit('user:identify', { userId, email })
socket.emit('room:join', 'notifications-room')
socket.emit('room:leave', 'notifications-room')
```

### Server → Client

```javascript
// User events
socket.on('registration-confirmed', (data) => {})
socket.on('waitlist-promotion', (data) => {})
socket.on('event-reminder', (data) => {})
socket.on('attendance-marked', (data) => {})

// Admin events
socket.on('admin:new-registration', (data) => {})
socket.on('admin:waitlist-promotion', (data) => {})
socket.on('admin:attendance-marked', (data) => {})
socket.on('admin:metrics-update', (data) => {})
```

## Event Emitter Events

### Emit Events

```javascript
import eventEmitter from '@/services/eventEmitterService.js';

// Registration confirmed
eventEmitter.emit('registration-confirmed', {
  userId, eventId, userEmail, userName, eventName,
  eventDate, eventTime, eventLocation, pushToken
});

// Waitlist promotion
eventEmitter.emit('waitlist-promotion', {
  userId, eventId, userEmail, userName, eventName,
  eventDate, eventTime, confirmationId, pushToken
});

// Event reminder
eventEmitter.emit('event-reminder', {
  userId, eventId, userEmail, userName, eventName,
  eventDate, eventTime, eventLocation, timeUntilEvent, pushToken
});

// Attendance marked
eventEmitter.emit('attendance-marked', {
  userId, eventId, userEmail, userName, eventName,
  eventDate, points, pushToken
});
```

## Email Templates

All emails are sent through SendGrid with professional HTML templates:

1. **Registration Confirmation**
   - Event details
   - Attendance instructions
   - Contact information

2. **Waitlist Promotion**
   - Congratulations message
   - Confirmation details
   - Event link

3. **Event Reminder**
   - Time until event
   - Event details
   - Add to calendar button

4. **Attendance Confirmation**
   - Points earned
   - Thank you message
   - Community message

## Push Notification Payload

```javascript
{
  title: 'Event Title',
  body: 'Notification message',
  icon: '/pwa-192x192.png',
  badge: '/pwa-192x192.png',
  tag: 'notification', // Prevents duplicate notifications
  data: {
    eventId: '123',
    type: 'registration', // registration, promotion, reminder, attendance
  },
  link: '/events/123' // Where to navigate when clicked
}
```

## PWA Features

### Service Worker
- **Cache Strategy**: Cache-first for static assets
- **Offline Support**: Fallback page for offline
- **Background Sync**: Sync data when back online
- **Push Handling**: Show notifications on device

### Manifest
- **Installable**: Add to home screen
- **Standalone Mode**: Full-screen app experience
- **Theme Colors**: Custom branding
- **Shortcuts**: Quick access to features

## Monitoring & Debugging

### Socket.IO Debugging

```javascript
// Enable Socket.IO debug logging
localStorage.debug = 'socket.io-client:*';
```

### Service Worker Debugging

```javascript
// Check service worker registration
navigator.serviceWorker.getRegistrations().then(registrations => {
  console.log('Service Workers:', registrations);
});

// View push subscriptions
navigator.serviceWorker.ready.then(registration => {
  return registration.pushManager.getSubscription();
}).then(subscription => {
  console.log('Push Subscription:', subscription);
});
```

### SSE Debugging

```javascript
const eventSource = new EventSource('/api/admin/stream');
eventSource.onmessage = (event) => {
  console.log('SSE Message:', event.data);
};
```

## Troubleshooting

### Push Notifications Not Working

1. Check VAPID keys are correct
2. Verify notification permission is granted
3. Service worker must be registered
4. Check browser push notification settings
5. Verify Firebase credentials

### Emails Not Sending

1. Verify SendGrid API key
2. Check sender email is verified
3. Review SendGrid logs
4. Check environment variable setup
5. Verify email templates exist

### Socket.IO Connection Issues

1. Check CORS configuration
2. Verify frontend URL matches env variable
3. Check firewall/proxy rules
4. Enable Socket.IO debug logging
5. Verify server is listening on correct port

### SSE Connection Timeout

1. Check Connection header is keep-alive
2. Verify heartbeat interval (default 30s)
3. Check client-side EventSource handling
4. Verify no proxies interfering with stream
5. Check browser SSE support

## Performance Considerations

- **Socket.IO**: ~100 concurrent connections per core, use clustering for scale
- **Push Notifications**: FCM handles ~1M notifications/second
- **SendGrid**: Up to 5000 emails per second with Pro plan
- **SSE**: ~1000 concurrent streams with Node.js
- **Service Worker**: Minimal overhead, runs in background

## Security

- Socket.IO connections validated with user identification
- Push subscriptions encrypted in transit
- Email addresses validated before sending
- Admin SSE stream requires authentication
- Service Worker HTTPS-only (except localhost)
- CORS configured for frontend URL only

## Testing

### Unit Tests
```bash
npm test -- --testPathPattern="real-time|push|socket"
```

### Integration Tests
```bash
npm test -- --testPathPattern="integration"
```

### Manual Testing
1. Register for event → Check registration email + push notification
2. Mark attendance → Check attendance confirmation
3. Admin dashboard → Check real-time activity feed
4. Offline mode → Service worker caches data
5. Install PWA → Home screen installation

## Dependencies Added

Backend:
- `socket.io@4.7.x` - WebSocket library
- `firebase-admin@12.x` - Firebase Cloud Messaging
- `@sendgrid/mail@8.x` - Email service (already present)

Frontend:
- `socket.io-client@4.7.x` - WebSocket client
- `workbox-window@7.x` - Service Worker management (PWA)

## Next Steps

1. Configure Firebase project and SendGrid account
2. Generate VAPID keys and add to environment
3. Set up email templates in SendGrid
4. Test Socket.IO connections
5. Monitor push notification delivery
6. Set up admin dashboard streaming
7. Deploy Service Worker to production
8. Monitor real-time feature usage

## Support & Documentation

- Socket.IO Docs: https://socket.io/docs/
- Firebase Docs: https://firebase.google.com/docs/cloud-messaging
- SendGrid Docs: https://docs.sendgrid.com/
- Web Push: https://web.dev/push-notifications-overview/
- PWA: https://web.dev/progressive-web-apps/
