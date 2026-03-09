# Frontend Reference — Project Structure, Design System, Examples

Load this when you need project structure details, code examples, or design system information.

---

## Tech Stack

- **Next.js 15** (App Router, Server Components where applicable)
- **TypeScript** (strict mode)
- **Ant Design 6** (primary component library)
- **Tailwind CSS 3.4** (utility classes for layout tweaks)
- **Zustand** (state management — auth store)
- **Recharts** (charting library)
- **Sonner** (toast notifications)
- **next-intl** (planned for i18n — not yet installed)

---

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `NEXT_PUBLIC_API_BASE_URL` | `http://localhost:8000` | Backend URL |
| `NEXT_PUBLIC_API_URL` | `http://localhost:8000` | Used in docker-compose |
| `NEXT_PUBLIC_OWNER_SUBDOMAIN` | `owner` | Subdomain for owner portal |
| `NEXT_PUBLIC_BASE_DOMAIN` | `localhost` | Base domain for subdomain routing |

---

## Design System

| Token | Value |
|---|---|
| Primary color | `#E84040` (red) |
| Background | `#F8F9FA` (light gray) |
| Sidebar width | 64px (collapsed, icon-only) |
| Border radius | 8-12px |
| Card class | `.crm-card` (white bg, border-radius, shadow) |
| Glass effect | `.glass-card` (translucent) |
| Pagination | 20 items per page |

---

## Auth & Routing

**Subdomain routing (middleware.ts):**
- `owner.localhost:3000` → `/owner/...` pages
- `company1.localhost:3000` → `/(company)/...` pages
- No subdomain → 400 error

**Permission system (store/auth.ts):**
- Role hierarchy: OWNER (4) > COMPANY_ADMIN (3) > COMPANY_MANAGER (2) > COMPANY_OPERATOR (1)
- `hasPermission(requiredRole)` — role-based check
- `hasPermissionString("contacts.read")` — fine-grained permission
- `ProtectedRoute` component enforces at page level

**Token management (lib/api.ts):**
- JWT stored in localStorage
- Automatic Bearer header attachment
- 401 → automatic refresh token attempt
- Refresh failure → redirect to login

---

## Project Structure

```
repos/s1p-frontend/src/
  app/
    (company)/                       → Route group: company user pages
      dashboard/page.tsx             → Operator dashboard (stat cards, sparklines, quick actions)
      contacts/
        page.tsx                     → Contact list (card grid, search, pagination)
        new/page.tsx                 → Create contact form
        [id]/page.tsx                → Contact detail (notes, activity, calls)
      leads/
        page.tsx                     → Lead list (card grid, status filter)
        new/page.tsx                 → Create lead form
        [id]/page.tsx                → Lead detail
      deals/
        page.tsx                     → Deal list (card grid, stage filter)
        new/page.tsx                 → Create deal form
        [id]/page.tsx                → Deal detail
      tasks/
        page.tsx                     → Task list (checkbox completion)
        new/page.tsx                 → Create task form
        [id]/page.tsx                → Task detail
      calls/
        page.tsx                     → Call history (filters: direction, outcome, date, "my calls")
        [id]/page.tsx                → Call detail with recording player
      users/
        page.tsx                     → Team management (card grid)
        invite/page.tsx              → Invite user form
        [id]/page.tsx                → User detail/edit
      analytics/page.tsx             → Team analytics dashboard
      profile/page.tsx               → User profile + password change
      settings/
        page.tsx                     → Settings hub
        permission-groups/page.tsx   → Permission group CRUD
        contract/page.tsx            → Contract status
      layout.tsx                     → Company layout wrapper (sidebar + content)
    owner/
      dashboard/page.tsx             → Platform stats
      companies/                     → Company CRUD pages
      contracts/                     → Contract management pages
      settings/page.tsx              → Owner settings
      profile/page.tsx               → Owner profile
      login/page.tsx                 → Owner login
      layout.tsx                     → Owner layout wrapper
    login/page.tsx                   → Company user login
    set-password/page.tsx            → Password setup
    forgot-password/page.tsx         → Password reset
    register/page.tsx                → Registration
    layout.tsx                       → Root layout
    page.tsx                         → Root redirect

  components/
    layout/
      sidebar.tsx                    → Collapsed 64px sidebar, icon-only, role-based nav, permission-filtered
      app-layout.tsx                 → Generic layout wrapper
    ui/                              → Reusable primitives (avatar, badge, button, card, checkbox, dialog, input, label, progress, select, tabs, textarea)
    auth/
      AuthLayout.tsx                 → Login/register page layout with illustrations
      ProtectedRoute.tsx             → Auth guard (requireAuth, requireOwner, requireRole, requiredPermission)
    icons/
      custom-icons.tsx               → PhoneIncoming, PhoneOutgoing SVGs
    illustrations/                   → WelcomeCharacter, ErrorCharacter, SuccessCharacter, EmptyStateCharacter, SearchCharacter, AuthCharacter
    providers/
      AntdProvider.tsx               → Ant Design theme configuration (primary #E84040)

  lib/
    api.ts                           → ApiClient singleton (694 lines, ~80 methods, JWT auto-attach, 401 refresh)
    constants.ts                     → Display labels, colors, status enums (182 lines)
    subdomain.ts                     → Subdomain parsing utility
    toast.ts                         → Toast notification helpers (success, error, info)
    utils.ts                         → Utility functions (getErrorMessage, formatting)
    icons.ts                         → Icon mappings

  store/
    auth.ts                          → Zustand: user, userType, permissions, isAuthenticated, hasPermission(), initAuth()

  types/
    api.ts                           → Hand-written API types (311 lines)
    generated.ts                     → Auto-generated from OpenAPI spec

  middleware.ts                      → Subdomain routing (owner.* vs company.*)
  globals.css                        → Global styles + Tailwind imports
```

---

## Code Examples

### API Client Method
```typescript
// In src/lib/api.ts
async getNewThing(params?: NewThingParams): Promise<NewThingResponse> {
  const response = await this.request<NewThingResponse>('/api/v1/company/new-thing/', {
    params: params as Record<string, string>,
  });
  return response;
}
```

### List Page Pattern
```typescript
'use client';

import { useState, useEffect } from 'react';
import { apiClient } from '@/lib/api';
import { useAuthStore } from '@/store/auth';
import ProtectedRoute from '@/components/auth/ProtectedRoute';

export default function FeatureListPage() {
  const [items, setItems] = useState<Item[]>([]);
  const [loading, setLoading] = useState(true);
  const [currentPage, setCurrentPage] = useState(1);
  const [total, setTotal] = useState(0);
  const [search, setSearch] = useState('');
  const { hasPermissionString } = useAuthStore();

  const fetchItems = async () => {
    try {
      setLoading(true);
      const data = await apiClient.getItems({ page: currentPage, limit: 20, search });
      setItems(data.items);
      setTotal(data.total);
    } catch (error) {
      toast.error(getErrorMessage(error));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => { fetchItems(); }, [currentPage, search]);

  return (
    <ProtectedRoute requiredPermission="feature.read">
      <div style={{ padding: '24px' }}>
        {/* Header with title + action button */}
        {/* Search/filter bar */}
        {/* Loading state → Spin component */}
        {/* Empty state → EmptyStateCharacter */}
        {/* Data state → Card grid or table */}
        {/* Pagination → Ant Design Pagination */}
      </div>
    </ProtectedRoute>
  );
}
```

### Card Grid Layout
```typescript
<div style={{
  display: 'grid',
  gridTemplateColumns: 'repeat(auto-fill, minmax(350px, 1fr))',
  gap: '16px'
}}>
  {items.map(item => (
    <div key={item.id} className="crm-card" style={{ padding: '20px' }}>
      {/* Card content */}
    </div>
  ))}
</div>
```

### Form Pattern
```typescript
import { Form, Input, Select, Button } from 'antd';

export default function CreateFeaturePage() {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const handleSubmit = async (values: CreateFeatureData) => {
    try {
      setLoading(true);
      await apiClient.createFeature(values);
      toast.success('Created successfully');
      router.push('/feature');
    } catch (error) {
      toast.error(getErrorMessage(error));
    } finally {
      setLoading(false);
    }
  };

  return (
    <ProtectedRoute requiredPermission="feature.create">
      <div style={{ padding: '24px', maxWidth: '600px' }}>
        <Form form={form} layout="vertical" onFinish={handleSubmit}>
          <Form.Item name="title" label="Title" rules={[{ required: true }]}>
            <Input placeholder="Enter title" />
          </Form.Item>
          <Button type="primary" htmlType="submit" loading={loading}>
            Create
          </Button>
        </Form>
      </div>
    </ProtectedRoute>
  );
}
```

### Permission Check
```typescript
const { hasPermissionString, isAdmin } = useAuthStore();

{hasPermissionString('users.invite') && (
  <Button type="primary" onClick={() => router.push('/users/invite')}>
    Invite User
  </Button>
)}
```

### Loading & Empty States
```typescript
if (loading) return <div style={{ textAlign: 'center', padding: '60px' }}><Spin size="large" /></div>;

if (!loading && items.length === 0) return (
  <div style={{ textAlign: 'center', padding: '60px' }}>
    <EmptyStateCharacter />
    <p>No items found</p>
  </div>
);
```

### Component File Structure
```typescript
// src/components/[category]/[ComponentName].tsx
'use client';

import { ... } from 'antd';

interface ComponentNameProps {
  // Explicit props, no `any` types
}

export default function ComponentName({ prop1, prop2 }: ComponentNameProps) {
  // ...
  return (...);
}
```

### Testing Pattern
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { vi } from 'vitest';

vi.mock('@/lib/api', () => ({
  apiClient: {
    getContacts: vi.fn().mockResolvedValue({ items: [], total: 0 }),
  },
}));

vi.mock('@/store/auth', () => ({
  useAuthStore: () => ({
    isAuthenticated: true,
    hasPermissionString: () => true,
    user: { id: '1', role: 'COMPANY_ADMIN' },
  }),
}));

vi.mock('next/navigation', () => ({
  useRouter: () => ({ push: vi.fn() }),
  usePathname: () => '/contacts',
}));

describe('ContactsPage', () => {
  it('renders contact list', async () => {
    render(<ContactsPage />);
    await waitFor(() => {
      expect(screen.getByText('Contacts')).toBeInTheDocument();
    });
  });
});
```

### Ant Design Components
- **Buttons:** `<Button type="primary">`, `<Button type="default">`, `<Button danger>`
- **Tags for status:** `<Tag color="green">Active</Tag>`
- **Select for filters:** `<Select options={...} onChange={...} allowClear />`
- **Pagination:** `<Pagination current={page} total={total} pageSize={20} onChange={setPage} />`
- **Spin for loading:** `<Spin size="large" />`
- **Modal for confirmations:** `Modal.confirm({ title: '...', onOk: () => ... })`
- **Icons:** Import from `@ant-design/icons` — never create custom icons when Ant has one
