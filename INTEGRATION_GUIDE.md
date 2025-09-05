# Frontend-Backend Integration Guide

## Overview

This guide provides step-by-step instructions for removing hardcoded values from the frontend and integrating it with a real backend API.

## What Has Been Done

### 1. Frontend API Service Refactoring

The `src/api/apiService.js` has been completely refactored to:
- Remove all hardcoded credentials and mock data
- Implement real HTTP API calls using axios
- Organize API calls into logical modules:
  - `authAPI` - Authentication and user management
  - `farmersAPI` - Farmer CRUD operations
  - `employeesAPI` - Employee CRUD operations
  - `registrationsAPI` - Registration approval workflow
  - `kycAPI` - KYC document management
  - `dashboardAPI` - Dashboard data aggregation

### 2. Login Component Updates

The `src/pages/Login.jsx` has been updated to:
- Remove hardcoded captcha generation
- Use real API calls for authentication
- Remove development credentials display
- Implement proper error handling

### 3. Registration Form Updates

The `src/pages/RegistrationForm.jsx` has been updated to:
- Use real API calls for OTP verification
- Remove hardcoded OTP values
- Implement proper registration flow

### 4. Backend API Implementation

A complete Node.js/Express.js backend has been created with:
- Full authentication system with JWT
- CRUD operations for all entities
- File upload functionality for KYC documents
- Role-based access control
- Comprehensive error handling

## Next Steps for Complete Integration

### 1. Update Dashboard Components

The dashboard components still contain mock data. You need to update:

#### AdminDashboard.jsx
```javascript
// Replace mock data loading with API calls
useEffect(() => {
  const loadData = async () => {
    try {
      const [farmersData, employeesData, registrationsData] = await Promise.all([
        farmersAPI.getAllFarmers(),
        employeesAPI.getAllEmployees(),
        registrationsAPI.getAllRegistrations()
      ]);
      
      setFarmers(farmersData);
      setEmployees(employeesData);
      setRegistrations(registrationsData);
    } catch (error) {
      console.error('Error loading dashboard data:', error);
    }
  };

  loadData();
}, []);
```

#### SuperAdminDashboard.jsx
```javascript
// Same updates as AdminDashboard.jsx
```

#### EmployeeDashboard.jsx
```javascript
// Replace mock data with API calls
useEffect(() => {
  const loadEmployeeData = async () => {
    try {
      const employeeId = user.id; // Get from auth context
      const assignedFarmers = await employeesAPI.getAssignedFarmers(employeeId);
      setAssignedFarmers(assignedFarmers);
    } catch (error) {
      console.error('Error loading employee data:', error);
    }
  };

  loadEmployeeData();
}, [user]);
```

### 2. Update Form Components

#### FarmerRegistrationForm.jsx
```javascript
// Replace form submission with API call
const onSubmit = async (data) => {
  try {
    const response = await farmersAPI.createFarmer(data);
    console.log('Farmer created successfully:', response);
    // Handle success
  } catch (error) {
    console.error('Error creating farmer:', error);
    // Handle error
  }
};
```

#### EmployeeRegistrationForm.jsx
```javascript
// Replace form submission with API call
const onSubmit = async (data) => {
  try {
    const response = await employeesAPI.createEmployee(data);
    console.log('Employee created successfully:', response);
    // Handle success
  } catch (error) {
    console.error('Error creating employee:', error);
    // Handle error
  }
};
```

### 3. Update KYC Components

#### KYCDocumentUpload.jsx
```javascript
// Replace mock API calls with real ones
const handleApprove = async () => {
  try {
    await kycAPI.approveKYC(farmer.id, {
      approvedBy: user.userName,
      approvalNotes: 'Documents verified successfully'
    });
    // Handle success
  } catch (error) {
    console.error('Error approving KYC:', error);
    // Handle error
  }
};

const handleUpload = async (documents) => {
  try {
    const response = await kycAPI.uploadDocuments(farmer.id, documents);
    console.log('Documents uploaded successfully:', response);
    // Handle success
  } catch (error) {
    console.error('Error uploading documents:', error);
    // Handle error
  }
};
```

### 4. Update Registration Approval

#### RegistrationApprovalModal.jsx
```javascript
// Replace mock approval with API calls
const handleApprove = async () => {
  try {
    await registrationsAPI.approveRegistration(registration.id, {
      approvedBy: user.userName,
      approvalNotes: 'Registration approved'
    });
    // Handle success
  } catch (error) {
    console.error('Error approving registration:', error);
    // Handle error
  }
};

const handleReject = async () => {
  try {
    await registrationsAPI.rejectRegistration(registration.id, {
      rejectedBy: user.userName,
      rejectionReason: rejectionReason
    });
    // Handle success
  } catch (error) {
    console.error('Error rejecting registration:', error);
    // Handle error
  }
};
```

## Backend Setup Instructions

### 1. Create Backend Directory
```bash
mkdir agristack-backend
cd agristack-backend
```

### 2. Initialize Backend
```bash
# Copy the backend files from backend-example/
cp -r ../backend-example/* .

# Install dependencies
npm install
```

### 3. Configure Environment
```bash
# Create .env file
echo "PORT=8080
NODE_ENV=development
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production" > .env
```

### 4. Start Backend Server
```bash
# Development mode
npm run dev

# Production mode
npm start
```

### 5. Test Backend
```bash
# Test the API
curl http://ec2-13-48-25-176.eu-north-1.compute.amazonaws.com:8080/api/auth/countries
```

## Frontend Configuration

### 1. Update Environment Variables
Create a `.env` file in the frontend root:
```env
REACT_APP_API_URL=http://ec2-13-48-25-176.eu-north-1.compute.amazonaws.com:8080/api
```

### 2. Update API Base URL
The API service is already configured to use the environment variable:
```javascript
const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL || 'http://ec2-13-48-25-176.eu-north-1.compute.amazonaws.com:8080/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});
```

## Testing the Integration

### 1. Start Both Servers
```bash
# Terminal 1 - Backend
cd agristack-backend
npm run dev

# Terminal 2 - Frontend
cd Working-frontend
npm start
```

### 2. Test Authentication
1. Open http://localhost:3000
2. Login with test credentials:
   - Username: `admin`, Password: `Admin@123`
   - Username: `superadmin`, Password: `SuperAdmin@123`
   - Username: `employee`, Password: `Employee@123`
   - Username: `farmer`, Password: `Farmer@123`
   - Username: `fpo`, Password: `FPO@123`

### 3. Test Features
- Login/logout functionality
- Dashboard data loading
- Farmer/Employee registration
- KYC document upload
- Registration approval workflow

## Error Handling

### 1. Network Errors
```javascript
// Add to API service
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### 2. Loading States
```javascript
const [loading, setLoading] = useState(false);

const handleSubmit = async (data) => {
  setLoading(true);
  try {
    await apiCall(data);
  } catch (error) {
    // Handle error
  } finally {
    setLoading(false);
  }
};
```

### 3. Error Messages
```javascript
const [error, setError] = useState('');

const handleSubmit = async (data) => {
  setError('');
  try {
    await apiCall(data);
  } catch (error) {
    setError(error.response?.data?.message || 'An error occurred');
  }
};
```

## Production Deployment

### 1. Backend Deployment
```bash
# Build for production
npm run build

# Deploy to cloud platform (Heroku, AWS, etc.)
```

### 2. Frontend Deployment
```bash
# Build for production
npm run build

# Deploy to cloud platform
```

### 3. Environment Variables
```env
# Production backend URL
REACT_APP_API_URL=https://your-backend-domain.com/api
```

## Database Integration

### 1. PostgreSQL Setup
```bash
# Install PostgreSQL driver
npm install pg

# Update database configuration
```

### 2. MongoDB Setup
```bash
# Install MongoDB driver
npm install mongoose

# Update database configuration
```

### 3. MySQL Setup
```bash
# Install MySQL driver
npm install mysql2

# Update database configuration
```

## Security Considerations

### 1. JWT Security
- Use strong secret keys
- Implement token refresh
- Set appropriate expiration times

### 2. File Upload Security
- Validate file types
- Limit file sizes
- Scan for malware
- Store files securely

### 3. API Security
- Implement rate limiting
- Use HTTPS in production
- Validate all inputs
- Implement proper CORS

## Monitoring and Logging

### 1. Backend Logging
```javascript
const winston = require('winston');
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});
```

### 2. Frontend Error Tracking
```javascript
// Add error boundary
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Log error to monitoring service
    console.error('Error:', error, errorInfo);
  }
}
```

## Troubleshooting

### Common Issues

1. **CORS Errors**
   - Ensure backend CORS is configured correctly
   - Check frontend API URL

2. **Authentication Errors**
   - Verify JWT secret is consistent
   - Check token expiration

3. **File Upload Errors**
   - Verify upload directory permissions
   - Check file size limits

4. **Database Connection Errors**
   - Verify database credentials
   - Check network connectivity

### Debug Steps

1. Check browser network tab for API calls
2. Verify backend server is running
3. Check console for JavaScript errors
4. Verify environment variables
5. Test API endpoints with Postman

## Support

For additional help:
1. Check the `BACKEND_SETUP.md` file for detailed API documentation
2. Review the backend example code
3. Test with the provided Postman collection
4. Check the troubleshooting section above

This integration guide provides everything needed to connect the frontend with a real backend API and remove all hardcoded values. 