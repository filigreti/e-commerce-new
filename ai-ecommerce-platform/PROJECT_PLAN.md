# 🛍️ AI-Powered Multi-Vendor E-Commerce Platform - Implementation Plan

## 📋 Project Overview
A full-featured e-commerce marketplace similar to ASOS with AI virtual try-on capabilities, multi-vendor support, and automated seller onboarding.

---

## 🏗️ Phase 1: Foundation & Setup

### 1.1 Tech Stack
```
Frontend:
- Next.js 15 (App Router)
- React 19
- TypeScript
- Tailwind CSS v4
- shadcn/ui components
- pnpm package manager

Backend:
- Next.js API Routes
- Neon PostgreSQL (Serverless)
- Drizzle ORM (lighter & faster than Prisma)
- Vercel AI SDK (for AI features)

AI/ML:
- Google Gemini 2.5 Flash Image (virtual try-on)
- AI Gateway (monitoring & cost control)

Auth:
- NextAuth.js v5 (supports multi-role auth)

Storage:
- Vercel Blob / Uploadthing (product images)
- Image optimization with Next.js Image
```

### 1.2 Project Structure
```
/ai-ecommerce-platform
├── /app
│   ├── /(auth)              # Auth routes (login, register)
│   ├── /(customer)          # Customer-facing routes
│   │   ├── /men             # Men's section
│   │   ├── /women           # Women's section
│   │   ├── /brands          # All brands directory
│   │   ├── /brand/[slug]    # Individual brand page
│   │   ├── /product/[id]    # Product details + AI try-on
│   │   ├── /cart
│   │   ├── /checkout
│   │   └── /orders
│   ├── /(seller)            # Seller dashboard
│   │   ├── /dashboard
│   │   ├── /products        # Manage products
│   │   ├── /orders
│   │   └── /analytics
│   ├── /(admin)             # Admin panel
│   │   ├── /sellers         # Vet sellers
│   │   ├── /products        # Moderate products
│   │   └── /analytics
│   └── /api
│       ├── /auth            # Authentication
│       ├── /products        # Product CRUD
│       ├── /sellers         # Seller management
│       ├── /ai-tryon        # AI virtual try-on
│       └── /upload          # File uploads
├── /components
│   ├── /ui                  # shadcn components
│   ├── /customer            # Customer components
│   ├── /seller              # Seller components
│   └── /shared              # Shared components
├── /lib
│   ├── /db                  # Drizzle schema & config
│   ├── /ai                  # AI integration utilities
│   └── /utils               # Helpers
└── /drizzle                 # Migrations
```

---

## 🗄️ Phase 2: Database Schema

### 2.1 Core Tables (Drizzle Schema)

#### Users Table (multi-role)
```typescript
users {
  id: uuid (PK)
  email: string (unique)
  password: string (hashed)
  name: string
  role: enum ('customer', 'seller', 'admin')
  avatar: string (nullable)
  createdAt: timestamp
  updatedAt: timestamp
}
```

#### Seller Profiles (Brands)
```typescript
sellers {
  id: uuid (PK)
  userId: uuid (FK -> users)
  businessName: string (brand name)
  brandSlug: string (unique, for brand pages /brand/nike)
  businessType: enum ('individual', 'business')
  phone: string
  address: json
  taxId: string (nullable)
  status: enum ('pending', 'approved', 'rejected', 'suspended')
  verificationDocs: json (nullable)
  socialMedia: json (instagram, whatsapp)
  logo: string (nullable, brand logo)
  banner: string (nullable, brand page banner)
  description: text (nullable, about the brand)
  createdAt: timestamp
  approvedAt: timestamp (nullable)
  approvedBy: uuid (FK -> users, nullable)
}
```

**Note:** Sellers = Brands in this platform. Each seller has their own:
- Brand page: `/brand/{brandSlug}` (e.g., `/brand/nike`)
- Admin dashboard to manage their products
- Brand identity (logo, banner, description)
- Products linked to their seller account

#### Categories
```typescript
categories {
  id: uuid (PK)
  name: string
  slug: string (unique)
  gender: enum ('men', 'women', 'unisex')
  parentId: uuid (FK -> categories, nullable)
  image: string (nullable)
  order: int
}
```

##### Category Structure (Main Categories → Subcategories)

**MEN:**
- **Clothing** (Main)
  - T-Shirts & Vests
  - Shirts
  - Jeans
  - Trousers
  - Shorts
  - Jackets & Coats
  - Hoodies & Sweatshirts
  - Suits & Blazers
  - Native Wears (Agbada, Kaftan, Senator, Dashiki)
  - Activewear

- **Shoes** (Main)
  - Trainers
  - Boots
  - Formal Shoes
  - Sandals & Slides
  - Casual Shoes

- **Accessories** (Main)
  - Watches
  - Bags & Backpacks
  - Belts
  - Hats & Caps
  - Sunglasses
  - Wallets

**WOMEN:**
- **Clothing** (Main)
  - Dresses
  - Tops & T-Shirts
  - Jeans
  - Trousers
  - Skirts
  - Jackets & Coats
  - Jumpsuits & Playsuits
  - Knitwear
  - Native Wears (Gele, Iro & Buba, Kaftan, Ankara Styles)
  - Activewear

- **Shoes** (Main)
  - Heels
  - Boots
  - Trainers
  - Flats
  - Sandals

- **Accessories** (Main)
  - Bags & Purses
  - Jewelry
  - Belts
  - Hats
  - Sunglasses
  - Scarves

**UNISEX:**
- **Accessories**
  - Watches
  - Sunglasses
  - Bags

#### Products
```typescript
products {
  id: uuid (PK)
  sellerId: uuid (FK -> sellers)
  categoryId: uuid (FK -> categories)
  name: string
  slug: string (unique)
  description: text
  price: decimal
  compareAtPrice: decimal (nullable)
  gender: enum ('men', 'women', 'unisex')
  status: enum ('draft', 'pending', 'active', 'rejected')
  aiTryonEnabled: boolean (default: true)
  createdAt: timestamp
  updatedAt: timestamp
}
```

#### Product Images
```typescript
productImages {
  id: uuid (PK)
  productId: uuid (FK -> products)
  url: string
  altText: string (nullable)
  order: int
  isPrimary: boolean
}
```

#### Product Variants (sizes, colors)
```typescript
productVariants {
  id: uuid (PK)
  productId: uuid (FK -> products)
  sku: string (unique)
  size: string (nullable)
  color: string (nullable)
  stock: int
  price: decimal (nullable) // Override product price
}
```

#### Orders
```typescript
orders {
  id: uuid (PK)
  customerId: uuid (FK -> users)
  orderNumber: string (unique)
  status: enum ('pending', 'processing', 'shipped', 'delivered', 'cancelled')
  subtotal: decimal
  tax: decimal
  shipping: decimal
  total: decimal
  shippingAddress: json
  createdAt: timestamp
}
```

#### Order Items
```typescript
orderItems {
  id: uuid (PK)
  orderId: uuid (FK -> orders)
  productId: uuid (FK -> products)
  variantId: uuid (FK -> productVariants, nullable)
  sellerId: uuid (FK -> sellers)
  quantity: int
  price: decimal
  total: decimal
}
```

#### AI Try-On Sessions
```typescript
aiTryonSessions {
  id: uuid (PK)
  userId: uuid (FK -> users, nullable)
  productId: uuid (FK -> products)
  userPhotoUrl: string
  generatedImageUrl: string
  prompt: text
  modelUsed: string
  tokensUsed: int
  cost: decimal
  createdAt: timestamp
}
```

#### Seller Vetting
```typescript
sellerVerifications {
  id: uuid (PK)
  sellerId: uuid (FK -> sellers)
  reviewedBy: uuid (FK -> users, nullable)
  status: enum ('pending', 'approved', 'rejected')
  notes: text (nullable)
  documents: json
  createdAt: timestamp
  reviewedAt: timestamp (nullable)
}
```

---

## 🔐 Phase 3: Authentication & Authorization

### 3.1 User Roles & Permissions

**Customer:**
- Browse products
- Use AI try-on
- Add to cart, checkout
- View orders

**Seller:**
- Manage products (CRUD)
- Upload products (single/batch)
- View analytics & sales
- Manage orders

**Admin:**
- Vet sellers
- Moderate products
- Platform analytics
- User management

### 3.2 Auth Flow
1. Registration → Choose role (customer/seller)
2. If seller → Additional business info → Pending approval
3. Login → Role-based redirect
4. Protected routes with middleware

---

## 🎨 Phase 4: Customer Frontend

### 4.1 Homepage
- Hero section with gender toggle (Men/Women - default: Men)
- Featured products (filtered by selected gender)
- Category grid
- AI Try-On CTA

### 4.2 Product Listing (/men, /women)
- Filter sidebar (categories, price, size, color, brands)
- Sort options (newest, price, popularity)
- Grid/List view toggle
- Pagination/Infinite scroll
- Gender-specific products

### 4.3 Brand Pages
- **Brand Directory (/brands)**
  - Grid of all approved sellers/brands
  - Brand logos and names
  - Product count per brand
  - Filter by gender

- **Individual Brand Page (/brand/{brandSlug})**
  - Brand banner & logo
  - Brand description/about
  - All products from this brand
  - Filter by category & gender
  - Link to brand social media

### 4.4 Product Detail Page
- Image gallery
- Product info (name, price, description)
- Size/color selector
- Stock status
- AI Virtual Try-On button → Opens modal
- Add to cart
- Brand/Seller info (clickable to brand page)

### 4.5 AI Virtual Try-On Modal
1. User uploads photo (drag/drop or click)
2. Shows loading with progress
3. Generates AI image using Gemini (like current codebase)
4. Display result with download option
5. Save to session for logged-in users

---

## 🏪 Phase 5: Seller Features

### 5.1 Seller Onboarding (Brand Registration)
1. **Registration → Seller application form**
   - Business/Brand details (brand name, slug)
   - Brand identity (logo, banner, description)
   - Tax info
   - Social media (Instagram/WhatsApp for future AI scraping)
   - Upload verification docs

2. **Pending review** → Admin vets seller

3. **Approved** → Access to seller dashboard + Brand page goes live

### 5.2 Product Management

**Single Product Upload:**
- Manual form (name, description, price, category, gender)
- Image upload (multiple images)
- Variants (size, color, stock)

**Batch Upload:**
- CSV template download
- Upload CSV with product data
- Bulk image upload (zip file)
- Map images to products

**Product Status:**
- Draft → Edit freely
- Pending → Awaiting approval
- Active → Live on store
- Rejected → Admin feedback

### 5.3 Dashboard
- Sales analytics
- Recent orders
- Product performance
- Inventory alerts

---

## 👨‍💼 Phase 6: Admin Panel

### 6.1 Seller Vetting
- List of pending sellers
- View application details
- Review verification docs
- Approve/Reject with notes
- Suspend existing sellers

### 6.2 Product Moderation
- Review pending products
- Approve/Reject
- Flag inappropriate content

### 6.3 Platform Analytics
- Total sales
- Active sellers
- Product stats
- AI try-on usage

---

## 🤖 Phase 7: AI Integration

### 7.1 Virtual Try-On (Immediate)
- Reuse existing Gemini 2.5 Flash Image code
- Product-specific prompts for clothing types
- Save generated images
- Track AI costs per session

### 7.2 Future AI Features (Phase 2)
- Instagram catalog scraper (AI extracts product info from posts)
- WhatsApp Business integration (AI processes catalog messages)
- Smart categorization (AI suggests categories for products)
- Size recommendation (AI predicts best size from user photo)

---

## 🚀 Phase 8: Implementation Order

### Week 1: Foundation
1. ✅ Initialize Next.js 15 + pnpm + shadcn
2. ✅ Setup Drizzle + Neon DB
3. ✅ Database schema & migrations
4. ✅ Auth system (NextAuth.js)

### Week 2: Core Features
5. ✅ Customer product browsing (men/women sections)
6. ✅ Product detail pages
7. ✅ Cart & checkout flow
8. ✅ AI virtual try-on integration

### Week 3: Seller Features
9. ✅ Seller registration & onboarding
10. ✅ Product upload (single)
11. ✅ Product upload (batch CSV)
12. ✅ Seller dashboard

### Week 4: Admin & Polish
13. ✅ Admin seller vetting
14. ✅ Product moderation
15. ✅ Analytics dashboards
16. ✅ Testing & deployment

---

## 📦 Key Features Summary

### ✅ Customer Experience:
- Gender-based navigation (Men/Women - default: Men)
- AI virtual try-on for all products
- Smooth shopping experience
- Cart & checkout

### ✅ Seller Experience:
- Easy onboarding with vetting process
- Single & batch product upload
- Sales analytics & dashboard
- Order management

### ✅ Admin Experience:
- Seller approval workflow
- Product moderation
- Platform oversight & analytics

### ✅ AI Features:
- Virtual try-on using Gemini 2.5 Flash Image (immediate)
- Social media catalog scraping (future phase)
- Smart product categorization (future phase)

---

## 🔧 Technical Decisions

### Why Drizzle over Prisma?
- Lighter weight and faster
- Better TypeScript inference
- SQL-like query builder
- Excellent for serverless (Neon)

### Why NextAuth.js v5?
- Built-in multi-role support
- Easy integration with Next.js 15
- Flexible provider options
- Session management

### Why Gemini 2.5 Flash Image?
- Already proven in existing codebase
- Multimodal (text + images)
- Fast response times (Flash variant)
- Cost-effective

### Why Vercel AI Gateway?
- Monitor AI costs
- Track usage
- Rate limiting
- Analytics

---

## 📝 Next Steps

Once this plan is approved, we will:

1. **Initialize the project** with Next.js 15, pnpm, and shadcn
2. **Setup the database** with Drizzle and Neon
3. **Create the authentication** system with NextAuth.js
4. **Build the customer-facing** features (product browsing, AI try-on)
5. **Implement seller** features (onboarding, product management)
6. **Build admin panel** (seller vetting, moderation)
7. **Deploy and test**

Ready to start building! 🚀
