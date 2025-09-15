# JALDISEVA - AI-Powered Telemedicine Platform

**Smart India Hackathon 2025 | Problem Statement 25018**

JALDISEVA is an innovative telemedicine platform designed to revolutionize healthcare access in rural India through AI-powered symptom checking, digital queue management, and seamless video consultations.

## 🎯 Problem Statement

**25018: Telemedicine and Digital Queue Management for Rural Healthcare**

Rural areas in India face significant challenges in accessing quality healthcare due to:
- Limited healthcare infrastructure
- Shortage of medical professionals
- Long travel distances to healthcare facilities
- Lack of digital healthcare solutions
- Language barriers and low digital literacy

## 🚀 Solution Overview

JALDISEVA addresses these challenges through:

### 🤖 AI-Powered Health Assistant
- **Symptom Analysis**: GPT-4 powered intelligent symptom checking
- **Voice Input**: Whisper API integration for voice-to-text in local languages
- **Health Recommendations**: Personalized health advice and doctor recommendations
- **Emergency Detection**: Automatic identification of critical symptoms

### 📱 Digital Queue Management
- **Real-time Queue Status**: Live updates on wait times and position
- **Smart Scheduling**: AI-optimized appointment scheduling
- **Priority Management**: Emergency case prioritization
- **Multi-location Support**: Queue management across multiple healthcare centers

### 🎥 Telemedicine Consultations
- **WebRTC Video Calls**: Low-bandwidth optimized video consultations
- **Screen Sharing**: Medical document and report sharing
- **Session Recording**: Consultation recordings for medical records
- **Multi-device Support**: Works on smartphones, tablets, and computers

### 💊 Digital Prescription Management
- **Voice-to-Prescription**: Doctors can dictate prescriptions using voice input
- **PDF Generation**: Automated prescription PDF creation
- **Pharmacy Integration**: Direct prescription sharing with local pharmacies
- **Medication Tracking**: Patient medication history and reminders

### 🏥 Multi-Role Platform
- **Patients**: Symptom checking, doctor search, queue joining, consultations
- **Doctors**: Queue management, consultations, prescription writing
- **Pharmacies**: Stock management, prescription fulfillment
- **Administrators**: System oversight and analytics

## 🛠️ Technology Stack

### Frontend
- **Framework**: Next.js 14 with App Router
- **Styling**: Tailwind CSS with custom healthcare theme
- **State Management**: Zustand for global state
- **Data Fetching**: React Query with optimistic updates
- **Real-time**: Socket.IO for live updates
- **PWA**: Offline-first architecture with service workers

### Backend
- **Runtime**: Node.js with Express.js
- **Database**: PostgreSQL with optimized indexing
- **Authentication**: JWT with role-based access control
- **AI Integration**: Google Gemini API
- **File Storage**: Local file system with upload management
- **Real-time**: Socket.IO for WebRTC signaling and notifications

### Infrastructure
- **Deployment**: Docker containers with PM2 process management
- **Web Server**: Nginx with SSL termination and load balancing
- **Database**: PostgreSQL with connection pooling
- **Monitoring**: Custom health checks and logging
- **Security**: Rate limiting, input validation, and HTTPS enforcement

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend API   │    │   Database      │
│   (Next.js)     │◄──►│   (Express.js)  │◄──►│  (PostgreSQL)   │
│                 │    │                 │    │                 │
│ • React Query   │    │ • JWT Auth      │    │ • User Data     │
│ • Socket.IO     │    │ • Google Gemini │    │ • Queue Data    │
│ • WebRTC        │    │ • File Upload   │    │ • Medical Data  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   External APIs │
                    │                 │
                    │ • Google Gemini │
                    └─────────────────┘
```

## 📋 Features

### For Patients
- ✅ **AI Symptom Checker** - Describe symptoms and get AI-powered health insights
- ✅ **Doctor Discovery** - Find and filter doctors by specialization and location
- ✅ **Queue Management** - Join digital queues and track wait times
- ✅ **Video Consultations** - Secure video calls with healthcare providers
- ✅ **Prescription Access** - View and download digital prescriptions
- ✅ **Health Dashboard** - Personal health overview and history

### For Doctors
- ✅ **Queue Management** - View and manage patient queues efficiently
- ✅ **Video Consultations** - Conduct remote consultations with patients
- ✅ **Voice Prescriptions** - Dictate prescriptions using voice input
- ✅ **Patient History** - Access comprehensive patient medical records
- ✅ **Schedule Management** - Manage availability and appointment slots
- ✅ **Analytics Dashboard** - Track consultation metrics and patient feedback

### For Pharmacies
- ✅ **Stock Management** - Track medicine inventory and expiry dates
- ✅ **Prescription Processing** - Receive and fulfill digital prescriptions
- ✅ **Availability Updates** - Real-time medicine availability for patients
- ✅ **Sales Analytics** - Track prescription fulfillment and revenue

### For Administrators
- ✅ **System Monitoring** - Comprehensive platform health and usage analytics
- ✅ **User Management** - Manage users, roles, and permissions
- ✅ **Content Management** - Update platform content and configurations
- ✅ **Report Generation** - Generate detailed usage and health outcome reports

## 🚀 Getting Started

### Prerequisites
- Node.js 18+ 
- PostgreSQL 15+
- Google Gemini API key

### Backend Setup

1. **Clone and navigate to backend**
   ```bash
   git clone <repository-url>
   cd jaldiseva/backend
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Environment configuration**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. **Database setup**
   ```bash
   # Create PostgreSQL database
   createdb jaldiseva
   
   # Run database migrations
   npm run db:setup
   ```

5. **Start development server**
   ```bash
   npm run dev
   ```

### Frontend Setup

1. **Navigate to frontend**
   ```bash
   cd ../frontend
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Environment configuration**
   ```bash
   cp .env.example .env.local
   # Edit .env.local with your configuration
   ```

4. **Start development server**
   ```bash
   npm run dev
   ```

### Environment Variables

#### Backend (.env)
```env
# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/jaldiseva
DB_HOST=localhost
DB_PORT=5432
DB_NAME=jaldiseva
DB_USER=your_username
DB_PASSWORD=your_password

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# Google Gemini AI Configuration
GEMINI_API_KEY=AIzaSyCs-pxODTsSkxMWTW2GCtbzD2ichb4BFnU

# Server Configuration
PORT=3001
FRONTEND_URL=http://localhost:3000

# File Upload Configuration
MAX_FILE_SIZE=10485760
UPLOAD_PATH=./uploads
```

#### Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=http://localhost:3001/api
NEXT_PUBLIC_SOCKET_URL=http://localhost:3001
NEXT_PUBLIC_APP_NAME=JALDISEVA
```

## 🚀 Deployment

For detailed production deployment instructions, see [DEPLOYMENT.md](./DEPLOYMENT.md).

### Quick Deployment Options

#### Option 1: Traditional VPS
- Ubuntu 20.04+ or CentOS 8+
- 4GB+ RAM, 50GB+ storage
- Follow the comprehensive deployment guide

#### Option 2: Docker Deployment
```bash
# Build and run with Docker Compose
docker-compose up -d
```

#### Option 3: Cloud Platforms
- **AWS**: EC2 + RDS + CloudFront
- **Google Cloud**: Compute Engine + Cloud SQL
- **Azure**: App Service + Azure Database
- **DigitalOcean**: Droplets + Managed Database

## 🧪 Development

### Project Structure
```
jaldiseva/
├── backend/
│   ├── config/          # Database and app configuration
│   ├── middleware/      # Authentication and validation
│   ├── routes/          # API endpoints
│   ├── uploads/         # File upload storage
│   └── server.js        # Main server file
├── frontend/
│   ├── src/
│   │   ├── app/         # Next.js app router pages
│   │   ├── components/  # Reusable UI components
│   │   └── lib/         # Utilities and API client
│   └── public/          # Static assets
├── database/
│   ├── schema.sql       # Database schema
│   └── seed.sql         # Sample data
└── docs/                # Documentation
```

### API Documentation

#### Authentication Endpoints
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `GET /api/auth/profile` - Get user profile
- `PUT /api/auth/profile` - Update user profile

#### Patient Endpoints
- `GET /api/patients` - List patients (doctors only)
- `GET /api/patients/search` - Search patients
- `PUT /api/patients/profile` - Update patient profile

#### Doctor Endpoints
- `GET /api/doctors` - List available doctors
- `GET /api/doctors/search` - Search doctors by specialization/location
- `PUT /api/doctors/availability` - Update doctor availability
- `GET /api/doctors/queue` - Get doctor's current queue

#### Queue Management
- `POST /api/queue/join` - Join a doctor's queue
- `GET /api/queue/status/:id` - Get queue status
- `PUT /api/queue/update/:id` - Update queue entry
- `DELETE /api/queue/leave/:id` - Leave queue

#### AI Integration
- `POST /api/ai/chat` - AI symptom checking
- `POST /api/ai/transcribe` - Voice-to-text transcription

#### Prescriptions
- `POST /api/prescriptions` - Create new prescription
- `GET /api/prescriptions` - List user prescriptions
- `GET /api/prescriptions/:id/pdf` - Download prescription PDF

### Testing

#### Backend Testing
```bash
cd backend
npm test                 # Run all tests
npm run test:unit       # Unit tests only
npm run test:integration # Integration tests only
npm run test:coverage   # Test coverage report
```

#### Frontend Testing
```bash
cd frontend
npm test                # Run all tests
npm run test:e2e       # End-to-end tests
npm run test:coverage  # Test coverage report
```

### Code Quality

#### Linting and Formatting
```bash
# Backend
npm run lint           # ESLint
npm run format         # Prettier

# Frontend
npm run lint           # Next.js ESLint
npm run format         # Prettier
```

## 🔒 Security

### Authentication & Authorization
- **JWT Tokens**: Secure stateless authentication
- **Role-based Access**: Patient, Doctor, Pharmacy, Admin roles
- **Password Hashing**: bcrypt with salt rounds
- **Session Management**: Secure token refresh mechanism

### Data Protection
- **Input Validation**: Comprehensive request validation
- **SQL Injection Prevention**: Parameterized queries
- **XSS Protection**: Content Security Policy headers
- **Rate Limiting**: API endpoint protection
- **File Upload Security**: Type and size validation

### Privacy Compliance
- **Data Encryption**: Sensitive data encryption at rest
- **Audit Logging**: Comprehensive activity logging
- **GDPR Compliance**: Data portability and deletion rights
- **HIPAA Considerations**: Healthcare data protection standards

## 📊 Performance

### Optimization Features
- **Database Indexing**: Optimized queries for large datasets
- **Caching Strategy**: Redis caching for frequently accessed data
- **Image Optimization**: Next.js automatic image optimization
- **Code Splitting**: Dynamic imports for reduced bundle size
- **CDN Integration**: Static asset delivery optimization

### Monitoring
- **Health Checks**: Automated system health monitoring
- **Performance Metrics**: Response time and throughput tracking
- **Error Tracking**: Comprehensive error logging and alerting
- **Usage Analytics**: User behavior and feature usage tracking

## 🌐 Accessibility

### Inclusive Design
- **WCAG 2.1 AA Compliance**: Web accessibility standards
- **Keyboard Navigation**: Full keyboard accessibility
- **Screen Reader Support**: ARIA labels and semantic HTML
- **High Contrast Mode**: Accessibility-friendly color schemes
- **Font Size Scaling**: Responsive typography

### Multi-language Support
- **Hindi Support**: Primary local language integration
- **English Interface**: Default international language
- **Voice Input**: Multi-language voice recognition
- **Cultural Adaptation**: Region-specific healthcare terminology

## 🤝 Contributing

### Development Workflow
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Contribution Guidelines
- Follow the existing code style and conventions
- Write comprehensive tests for new features
- Update documentation for API changes
- Ensure all tests pass before submitting PR
- Include detailed PR description with screenshots

### Code Review Process
- All PRs require at least one review
- Automated tests must pass
- Security review for sensitive changes
- Performance impact assessment
- Documentation updates verification

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

### Team
- **Frontend Development**: React/Next.js specialists
- **Backend Development**: Node.js and database experts
- **AI Integration**: Machine learning and NLP engineers
- **Healthcare Consultation**: Medical professionals and rural healthcare experts
- **UI/UX Design**: Healthcare-focused design specialists

### Technologies
- **Google Gemini**: AI capabilities
- **Next.js**: React framework for frontend development
- **PostgreSQL**: Reliable and scalable database solution
- **Socket.IO**: Real-time communication infrastructure
- **WebRTC**: Peer-to-peer video communication

### Inspiration
- **Rural Healthcare Workers**: Frontline heroes serving remote communities
- **Digital India Initiative**: Government's vision for digital healthcare
- **Open Source Community**: Collaborative development ecosystem
- **Healthcare Innovation**: Global telemedicine advancement efforts

## 📞 Support

### Documentation
- **API Documentation**: [API Docs](./docs/api.md)
- **Deployment Guide**: [Deployment](./DEPLOYMENT.md)
- **User Manual**: [User Guide](./docs/user-guide.md)
- **Developer Guide**: [Development](./docs/development.md)

### Community
- **GitHub Issues**: Bug reports and feature requests
- **Discussions**: Community questions and ideas
- **Wiki**: Comprehensive documentation and tutorials
- **Changelog**: Version history and update notes

### Contact
- **Technical Support**: Create a GitHub issue
- **Security Issues**: security@jaldiseva.com
- **General Inquiries**: info@jaldiseva.com
- **Partnership**: partnerships@jaldiseva.com

---

**Built with ❤️ for rural healthcare in India**

*Empowering healthcare access through technology innovation*
