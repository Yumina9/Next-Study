# Linking And Navigating

> Next.js에서 경로를 탐색하는 4가지 방법

1. <code>< Link ></code> Component
2. <code>useRouter</code> hook (Client Components)
3. <code>redirect</code> function (Server Components)
4. History Api

## <code>< Link ></code> Component

- HTML < a >태그를 확장하여 경로 간 프리페칭 및 클라이언트 측 탐색을 제공하는 내장 컴포넌트
- Next.js에서 경로 간을 탐색하는 1순위 방법
- <code>next/link</code>에서 가져와서 <code>href</code>로 넘김

```typescript
// app/page.tsx
import Link from "next/link";

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>;
}
```

### Examples

#### Linking to Dynamic Segments

- 동적 세그먼트에 연결할 때 템플릿 리터럴과 보간을 사용해 링크 목록을 생성할 수 있음

```typescript
// blog post를 생성하는 예제
// app/blog/PostList.ts
import Link from "next/link";

export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  );
}
```

#### Checking Active Links

- <code>usePathname()</code>를 사용해 링크가 활성화되었는지 확인할 수 있음
- 활성링크에 클래스를 추가하면 현재 <code>pathname</code>이 링크의 <code>href</code>와 일치하는지 확인할 수 있음

```typescript
// app/components/links.tsx
"use client";

import { usePathname } from "next/navigation";
import Link from "next/link";

export function Links() {
  const pathname = usePathname();

  return (
    <nav>
      <ul>
        <li>
          <Link className={`link ${pathname === "/" ? "active" : ""}`} href="/">
            Home
          </Link>
        </li>
        <li>
          <Link className={`link ${pathname === "/about" ? "active" : ""}`} href="/about">
            About
          </Link>
        </li>
      </ul>
    </nav>
  );
}
```

#### Scrolling to an <code>id</code>

- Next.js App Router의 기본 동작은 새 경로의 맨 위로 스크롤하거나 앞뒤로 탐색할 때 스크롤 위치를 유지하는 것
- 특정 id 탐색으로 스크롤하고싶다면, URL에 <code>#</code> 해시 링크를 추가하거나 <code>href</code> prop에 해시 링크를 전달하면 됨
- <code>< a ></code>요소가 <code>< Link ></code>에 렌더링 되기 때문에 가능

```typescript
<Link href="/dashboard#settings">Settings</Link>

//output
<a href="/dashboard#settings">Settings</a>
```

#### Disabling scroll restoration

- Next.js App Router의 기본 동작은 새 경로의 맨 위로 스크롤하거나 앞뒤로 탐색할 때 스크롤 위치를 유지하는 것
- 이 동작을 비활성화하려면 <code>< Link ></code>컴포넌트에 <code>scroll={false}</code>를 전달하거나 <code>router.push()</code> 또는 <code>router.replace()</code>에 <code>scroll: false</code>를 전달하면 됨

```typescript
// next/link
<Link href="/dashboard" scroll={false}>
  Dashboard
</Link>
```

```typescript
// useRouter
import { useRouter } from "next/navigation";

const router = useRouter();

router.push("/dashboard", { scroll: false });
```

## <code>useRouter()</code> hook

- <code>useRouter()</code> hook을 사용하면 Client Component의 경로를 프로그래밍 방식으로 변경할 수 있음

```typescript
// app/page.ts
"use client";

import { useRouter } from "next/navigation";

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push("/dashboard")}>
      Dashboard
    </button>
  );
}
```

> useRouter 사용에 대한 특정 요구 사항이 없는 한 < Link > 컴포넌트를 사용하여 경로 간 탐색을 추천

## <code>redirect</code> function

- 서버 컴포넌트의 경우 redirect 기능을 대신 사용

```typescript
// app/team/[id]/page.tsx
import { redirect } from "next/navigation";

async function fetchTeam(id: string) {
  const res = await fetch("https://...");
  if (!res.ok) return undefined;
  return res.json();
}

export default async function Profile({ params }: { params: { id: string } }) {
  const team = await fetchTeam(params.id);
  if (!team) {
    redirect("/login");
  }

  // ...
}
```

> redirect 알아두면 좋은점
>
> - 기본적으로 307(임시 리디렉션) 상태 코드를 반환
> - 서버작업에서 사용되는 경우 303(기타 참조)을 반환하는데 일반적으로 POST 요청의 결과로 성공 페이지로 리디렉션하는데 사용
> - 내부적으로 에러가 발생하므로 try/catch 블록 외부에서 호출해야 함
> - 렌더링 프로세스 중에 클라이언트 컴포넌트에서 호출할 수 있지만 이벤트 핸들러에서는 호출할 수 없는대신 useRouter훅 사용 가능
> - 렌더링 프로세스 전에 리디렉션하려면 next.config.js 또는 Middleware를 사용

## Using the native History API

- Next.js를 사용하면 기본 <code>window.history.pushState</code>와 <code>window.history.replaceState</code> 메서드를 사용해 페이지를 다시 로드하지 않고도 브라우저의 history 스택을 업데이트할 수 있음
- <code>pushState</code>와 <code>replaceState</code> 호출은 Next.js Router에 통합되어 <code>usePathname</code>와 <code>useSearchParams</code>와 동기화할 수 있음

#### <code>window.history.pushState</code>

- 브라우저의 history 스택에 새 항목을 추가하는데 사용
- 사용자는 이전 상태로 다시 탐색할 수 있음

```typescript
// 제품 목록 정렬 예시
"use client";

import { useSearchParams } from "next/navigation";

export default function SortProducts() {
  const searchParams = useSearchParams();

  function updateSorting(sortOrder: string) {
    const params = new URLSearchParams(searchParams.toString());
    params.set("sort", sortOrder);
    window.history.pushState(null, "", `?${params.toString()}`);
  }

  return (
    <>
      <button onClick={() => updateSorting("asc")}>Sort Ascending</button>
      <button onClick={() => updateSorting("desc")}>Sort Descending</button>
    </>
  );
}
```

#### <code>window.history.replaceState</code>

- 브라우저의 history 스택의 현재 항목을 바꾸는데 사용
- 사용자는 이전 상태로 다시 이동할 수 없음

```typescript
// application의 locale을 전환하는 예시
"use client";

import { usePathname } from "next/navigation";

export function LocaleSwitcher() {
  const pathname = usePathname();

  function switchLocale(locale: string) {
    // e.g. '/en/about' or '/fr/contact'
    const newPath = `/${locale}${pathname}`;
    window.history.replaceState(null, "", newPath);
  }

  return (
    <>
      <button onClick={() => switchLocale("en")}>English</button>
      <button onClick={() => switchLocale("fr")}>French</button>
    </>
  );
}
```
