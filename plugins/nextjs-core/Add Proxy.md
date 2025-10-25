---
description: Create or update Next.js 16 proxy with common patterns for request interception and modification
---

# Add Proxy

Create or update Next.js 16 proxy for authentication, redirects, and request handling.

## Instructions

1. Check if `proxy.ts` exists in the project root or `src` directory
2. If creating new proxy:
   - Create `proxy.ts` in the project root (same level as `app/` or `pages/`)
   - Import NextResponse and NextRequest from 'next/server'
   - Create and export the proxy function
   - Add matcher config for specific routes
3. Common proxy patterns to offer:
   - Quick authentication checks and redirects
   - A/B testing with rewrites
   - Request/Response header modifications
   - Geolocation-based routing
   - CORS header management
   - Rate limiting headers
   - Security headers (CSP, HSTS, etc.)
   - URL normalization and cleanup
4. Use proper TypeScript types
5. Include matcher configuration to optimize performance
6. Remember proxy limitations (no slow data fetching, no complex session management)

## Example Structure

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  // Proxy logic - keep it fast and efficient
}

export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * - public folder
     */
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

## Common Proxy Patterns

### Authentication & Authorization

```typescript
export function proxy(request: NextRequest) {
  const token = request.cookies.get('auth-token')
  const isAuthPage = request.nextUrl.pathname.startsWith('/login') ||
                     request.nextUrl.pathname.startsWith('/register')
  
  // Redirect unauthenticated users to login
  if (!token && !isAuthPage && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  // Redirect authenticated users away from auth pages
  if (token && isAuthPage) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }
  
  return NextResponse.next()
}
```

### A/B Testing with Rewrites

```typescript
export function proxy(request: NextRequest) {
  const bucket = request.cookies.get('ab-test-bucket')?.value || 
                 (Math.random() > 0.5 ? 'a' : 'b')
  
  const response = NextResponse.next()
  
  // Set cookie if not present
  if (!request.cookies.get('ab-test-bucket')) {
    response.cookies.set('ab-test-bucket', bucket, { maxAge: 60 * 60 * 24 * 30 })
  }
  
  // Rewrite to different pages based on bucket
  if (request.nextUrl.pathname === '/landing') {
    const url = request.nextUrl.clone()
    url.pathname = `/landing-${bucket}`
    return NextResponse.rewrite(url)
  }
  
  return response
}
```

### Security Headers

```typescript
export function proxy(request: NextRequest) {
  const response = NextResponse.next()
  
  // Add security headers
  response.headers.set('X-Frame-Options', 'DENY')
  response.headers.set('X-Content-Type-Options', 'nosniff')
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin')
  response.headers.set(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';"
  )
  response.headers.set(
    'Strict-Transport-Security',
    'max-age=31536000; includeSubDomains'
  )
  
  return response
}
```

### Geolocation-based Routing

```typescript
export function proxy(request: NextRequest) {
  const country = request.geo?.country || 'US'
  
  // Redirect to localized versions
  if (request.nextUrl.pathname === '/') {
    const locale = country === 'FR' ? 'fr' : 
                   country === 'DE' ? 'de' : 
                   country === 'ES' ? 'es' : 'en'
    
    return NextResponse.redirect(new URL(`/${locale}`, request.url))
  }
  
  return NextResponse.next()
}
```

### Rate Limiting Headers

```typescript
export function proxy(request: NextRequest) {
  const ip = request.ip || 'unknown'
  const response = NextResponse.next()
  
  // Add rate limit headers (actual limiting logic would be more complex)
  response.headers.set('X-RateLimit-Limit', '1000')
  response.headers.set('X-RateLimit-Remaining', '999')
  response.headers.set('X-RateLimit-Reset', new Date(Date.now() + 60000).toISOString())
  
  return response
}
```

## Proxy Best Practices

1. **Keep it fast**: Proxy runs on every matched request, so avoid slow operations
2. **Use appropriate matchers**: Don't run proxy on static assets
3. **Organize with modules**: Import complex logic from separate files
4. **Handle errors gracefully**: Always return a response, even on errors
5. **Test thoroughly**: Proxy affects all matched routes, so test edge cases

## Important Limitations

- **No fetch with cache options**: `options.cache`, `options.next.revalidate`, and `options.next.tags` have no effect
- **Not for data fetching**: Move data fetching to route handlers or server components
- **Not for session management**: Use dedicated session libraries in your application code
- **Single proxy file**: Only one `proxy.ts` per project (organize with imports if needed)

Ensure proxy logic is efficient and only handles quick request/response modifications.
