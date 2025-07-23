# Admin Platform

A comprehensive React-based admin platform with user management, balance control, transfer processing, and notification system built with modern web technologies.

![Admin Platform](https://images.pexels.com/photos/3184291/pexels-photo-3184291.jpeg?auto=compress&cs=tinysrgb&w=1200&h=400&fit=crop)

## 🚀 Features

- **Dashboard** - Real-time overview with key metrics and statistics
- **User Management** - Complete user account administration
- **Balance Management** - Control user balances and account freezing
- **Transfer Management** - Process and monitor transfers between users
- **Notification Center** - Send targeted notifications to users
- **Authentication** - Secure admin access with role-based permissions
- **Responsive Design** - Works perfectly on desktop and mobile devices

## 🛠️ Tech Stack

- **Frontend:** React 18 with TypeScript
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Backend:** Supabase (Database + Authentication)
- **Routing:** React Router DOM
- **Icons:** Lucide React
- **Deployment:** Netlify

## 📋 Prerequisites

Before you begin, ensure you have:

- Node.js 18+ installed
- A Supabase account
- A Netlify account (for deployment)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/admin-platform.git
cd admin-platform
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up Supabase

1. Create a new project at [supabase.com](https://supabase.com)
2. Go to Settings > API to get your project URL and anon key
3. Create a `.env` file in the root directory:

```env
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### 4. Set Up Database

Run these SQL commands in your Supabase SQL editor:

```sql
-- Create profiles table
CREATE TABLE profiles (
  id UUID REFERENCES auth.users ON DELETE CASCADE PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  role TEXT DEFAULT 'customer' CHECK (role IN ('admin', 'customer')),
  status TEXT DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'pending')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create user_balances table
CREATE TABLE user_balances (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE UNIQUE NOT NULL,
  balance DECIMAL(15,2) DEFAULT 0.00,
  currency TEXT DEFAULT 'USD',
  is_frozen BOOLEAN DEFAULT FALSE,
  last_updated TIMESTAMPTZ DEFAULT NOW()
);

-- Create notifications table
CREATE TABLE notifications (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE NOT NULL,
  title TEXT NOT NULL,
  message TEXT NOT NULL,
  type TEXT DEFAULT 'info' CHECK (type IN ('info', 'success', 'warning', 'error')),
  is_read BOOLEAN DEFAULT FALSE,
  sent_by UUID REFERENCES profiles(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create mock_transfers table
CREATE TABLE mock_transfers (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  from_user_id UUID REFERENCES profiles(id) NOT NULL,
  to_user_id UUID REFERENCES profiles(id) NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  currency TEXT DEFAULT 'USD',
  description TEXT,
  status TEXT DEFAULT 'completed' CHECK (status IN ('pending', 'completed', 'failed', 'cancelled')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create admin_activities table
CREATE TABLE admin_activities (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  admin_id UUID REFERENCES profiles(id) NOT NULL,
  action TEXT NOT NULL,
  target_user_id UUID REFERENCES profiles(id),
  details JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_balances ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE mock_transfers ENABLE ROW LEVEL SECURITY;
ALTER TABLE admin_activities ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
CREATE POLICY "Users can read own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Admins can read all profiles" ON profiles FOR SELECT USING (
  EXISTS (SELECT 1 FROM profiles WHERE id = auth.uid() AND role = 'admin')
);

-- Create functions
CREATE OR REPLACE FUNCTION update_user_balance(
  target_user_id UUID,
  new_balance DECIMAL,
  admin_id UUID
) RETURNS VOID AS $$
BEGIN
  UPDATE user_balances 
  SET balance = new_balance, last_updated = NOW()
  WHERE user_id = target_user_id;
  
  INSERT INTO admin_activities (admin_id, action, target_user_id, details)
  VALUES (admin_id, 'balance_update', target_user_id, 
    json_build_object('new_balance', new_balance));
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE FUNCTION process_mock_transfer(
  from_id UUID,
  to_id UUID,
  transfer_amount DECIMAL,
  transfer_description TEXT,
  admin_id UUID
) RETURNS VOID AS $$
BEGIN
  -- Update balances
  UPDATE user_balances SET balance = balance - transfer_amount WHERE user_id = from_id;
  UPDATE user_balances SET balance = balance + transfer_amount WHERE user_id = to_id;
  
  -- Create transfer record
  INSERT INTO mock_transfers (from_user_id, to_user_id, amount, description)
  VALUES (from_id, to_id, transfer_amount, transfer_description);
  
  -- Log admin activity
  INSERT INTO admin_activities (admin_id, action, details)
  VALUES (admin_id, 'transfer_created', 
    json_build_object('from_user', from_id, 'to_user', to_id, 'amount', transfer_amount));
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 5. Create Admin User

1. Sign up through your app or Supabase Auth
2. Update the user's role to admin in the profiles table:

```sql
UPDATE profiles SET role = 'admin' WHERE email = 'your-admin-email@example.com';
```

### 6. Run Development Server

```bash
npm run dev
```

Visit `http://localhost:3000` to see your admin platform!

## 🚀 Deployment

### Deploy to Netlify

1. Push your code to GitHub
2. Connect your GitHub repo to Netlify
3. Configure build settings:
   - **Build command:** `npm run build`
   - **Publish directory:** `dist`
   - **Functions directory:** (leave empty)

4. Add environment variables in Netlify:
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_ANON_KEY`

5. Deploy!

### Alternative: Manual Deployment

```bash
npm run build
# Upload the 'dist' folder to your hosting provider
```

## 📁 Project Structure

```
admin-platform/
├── public/
│   └── _redirects
├── src/
│   ├── components/
│   │   ├── Layout.tsx
│   │   ├── ProtectedRoute.tsx
│   │   └── TestConnection.tsx
│   ├── lib/
│   │   └── supabase.ts
│   ├── pages/
│   │   ├── admin/
│   │   │   ├── AdminDashboard.tsx
│   │   │   ├── UserManagement.tsx
│   │   │   ├── BalanceManagement.tsx
│   │   │   ├── TransferManagement.tsx
│   │   │   └── NotificationCenter.tsx
│   │   └── Dashboard.tsx
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── .env.example
├── package.json
└── README.md
```

## 🔧 Configuration

### Environment Variables

Create a `.env` file with:

```env
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### Supabase Configuration

The app expects these database tables:
- `profiles` - User profiles and roles
- `user_balances` - Account balances
- `notifications` - System notifications
- `mock_transfers` - Transfer records
- `admin_activities` - Admin action logs

## 🎯 Usage

### Admin Features

1. **Dashboard** - View system overview and statistics
2. **User Management** - Add, edit, suspend user accounts
3. **Balance Management** - Adjust user balances and freeze accounts
4. **Transfer Management** - Process transfers between users
5. **Notifications** - Send targeted messages to users

### User Roles

- **Admin** - Full access to all features
- **Customer** - Limited access (future implementation)

## 🛡️ Security

- Row Level Security (RLS) enabled on all tables
- Role-based access control
- Secure API endpoints through Supabase
- Environment variables for sensitive data

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

If you encounter any issues:

1. Check the [Issues](https://github.com/yourusername/admin-platform/issues) page
2. Create a new issue with detailed information
3. Include error messages and steps to reproduce

## 🙏 Acknowledgments

- [React](https://reactjs.org/) - UI Framework
- [Supabase](https://supabase.com/) - Backend as a Service
- [Tailwind CSS](https://tailwindcss.com/) - CSS Framework
- [Vite](https://vitejs.dev/) - Build Tool
- [Lucide](https://lucide.dev/) - Icon Library

---

**Made with ❤️ by [Your Name]**

⭐ Star this repo if you find it helpful!