# HeyLixNet - Demo Setup & Usage Guide

## 🚀 Quick Start (Demo Mode)

The app is currently running in **DEMO MODE** which means:
- No backend server required
- Demo credentials work immediately
- All features are functional with mock data
- Perfect for testing and demonstration

## 📱 Demo Credentials

```
Username: demo_user
Password: SecurePass@2025
```

Or use any username/password - demo mode accepts all credentials!

## ✨ Features Demonstrated

### 1. **Secure Authentication**
- Military-grade login interface
- VPN status indicator
- Device ID tracking
- Session management

### 2. **End-to-End Encrypted Messaging**
- Real-time chat interface
- Message encryption indicators
- Timestamp formatting
- Sender identification
- Group conversations

### 3. **Security Features**
- Screenshot prevention (configurable)
- Copy-paste restrictions (configurable)
- Export restrictions
- VPN tunnel protection
- Security logging

### 4. **Settings & Profile Management**
- User profile display
- VPN connection status
- Security feature toggles
- Notification preferences
- Biometric lock option
- Account verification status

### 5. **Groups Management**
- Multiple group support
- Unread message badges
- Last message preview
- Group navigation

## 🎯 How to Use

### Step 1: Login
1. Open the app
2. Enter demo credentials or any username/password
3. Click "Login"
4. You'll be redirected to the Messages screen

### Step 2: View Messages
1. See your groups list (Alpha Squadron, Command Center)
2. VPN status shown at top (Connected)
3. Tap any group to open chat

### Step 3: Send Messages
1. Type a message in the input field
2. Messages are automatically encrypted (E2E indicator shown)
3. Your messages appear on the right (blue)
4. Other messages appear on the left (white)
5. Lock icon indicates encryption

### Step 4: Explore Settings
1. Tap Settings tab
2. View your profile information
3. Check VPN status
4. See active security features
5. Toggle notifications and biometric lock
6. View app version and build info

### Step 5: Security Features
1. Screenshot prevention active (banner shown in chat)
2. All messages encrypted end-to-end
3. VPN tunnel protection active
4. Security logging enabled

## 🔧 Configuration

### Enable/Disable Security Features

Edit `constants/config.ts`:

```typescript
export const SECURITY_CONFIG = {
  SCREENSHOT_PREVENTION: true,  // Set to false to allow screenshots
  COPY_PASTE_RESTRICTION: true, // Set to false to allow copy/paste
  EXPORT_RESTRICTION: true,      // Set to false to allow exports
} as const;
```

### Switch to Real Backend

Edit `constants/config.ts`:

```typescript
export const API_CONFIG = {
  BASE_URL: 'http://your-backend-url:8000',
  DEMO_MODE: false,  // Set to false to use real backend
} as const;
```

## 📊 Demo Data

### Users
- **Demo User** (demo_user_001)
  - Role: Captain
  - Unit: Alpha Squadron
  - Service Number: SVC-2025-001

- **Command Center** (user_002)
- **Lt. Anderson** (user_003)

### Groups
1. **Alpha Squadron**
   - 3 unread messages
   - Last message: "Mission briefing at 0800"

2. **Command Center**
   - 0 unread messages
   - Last message: "Status update required"

### Sample Messages
- "Mission briefing scheduled for 0800 hours"
- "Roger that. All personnel confirmed."
- "Equipment check completed. Ready for deployment."

## 🔐 Security Architecture

### Encryption
- **Algorithm**: AES-256-GCM
- **Key Size**: 256 bits
- **End-to-End**: All messages encrypted before transmission

### VPN Protection
- Simulated VPN tunnel
- Token-based authentication
- Connection status monitoring
- Auto-refresh capability

### Authentication
- JWT-based tokens
- Refresh token rotation
- Device ID tracking
- Session timeout (1 hour)

### Logging
- All login attempts logged
- Message sending tracked
- Security events recorded
- User activity monitored

## 🎨 UI/UX Features

### Design Elements
- Clean, modern interface
- Military-inspired color scheme
- Intuitive navigation
- Responsive layouts
- Professional typography

### Color Palette
- Primary Blue: #1a73e8
- Success Green: #34a853
- Warning Yellow: #fbbc04
- Error Red: #ea4335
- Background: #f5f5f5

### Icons
- Lucide React Native icons
- Consistent sizing
- Semantic usage
- Accessibility support

## 📱 Platform Support

### iOS
- Full feature support
- Native feel
- Biometric authentication ready
- Push notifications ready

### Android
- Full feature support
- Material Design compliance
- Biometric authentication ready
- Push notifications ready

### Web
- Demo mode fully functional
- Responsive design
- Desktop-optimized
- Touch and mouse support

## 🚧 Limitations in Demo Mode

1. **No Real Backend**
   - Data not persisted
   - No real-time sync
   - Mock API responses

2. **No Real Encryption**
   - Simulated encryption
   - No actual key exchange
   - Demo purposes only

3. **No Real VPN**
   - Simulated VPN connection
   - No actual tunnel
   - Status indicators only

4. **Limited Data**
   - Pre-defined groups
   - Sample messages
   - Mock users

## 🔄 Switching to Production

To use with real Django backend:

1. Set up Django backend (see DJANGO_BACKEND_COMPLETE.md)
2. Update API_CONFIG.BASE_URL
3. Set DEMO_MODE to false
4. Configure real encryption keys
5. Set up VPN infrastructure
6. Deploy to production servers

## 📞 Support

For questions or issues:
- Check DJANGO_BACKEND_COMPLETE.md for backend setup
- Review code comments for implementation details
- Contact development team for assistance

## 🎓 Learning Resources

### Technologies Used
- **React Native**: Mobile framework
- **Expo**: Development platform
- **TypeScript**: Type safety
- **Django**: Backend framework
- **PostgreSQL**: Database
- **Redis**: Caching & real-time
- **JWT**: Authentication
- **WebSockets**: Real-time messaging

### Best Practices Demonstrated
- Clean architecture
- Type safety
- Error handling
- Security logging
- User experience
- Responsive design
- Code organization

---

**Status**: Demo Mode Active ✅
**Version**: 1.0.0
**Build**: 1
**Environment**: Development

**Classification**: UNCLASSIFIED (Demo Version)
**For**: Demonstration and Testing Purposes Only
