# Clean Code & Code Conventions Next.js

related links

-   [Clean Code JavaScript](https://github.com/FrontendEraspace/clean-code-javascript)
-   [ES Syntax](https://github.com/FrontendEraspace/es-2024-2023)

**File Structure and Organization**:
Basic folder structure, bisa disesuaikan sesuai kebutuhan dan skala aplikasi.

```
my-app/
├── components/
│   ├── Header/
│   │   ├── Header.js
│   │   └── Header.module.css
│   └── Footer/
│       ├── Footer.js
│       └── Footer.module.css
├── pages/
│   ├── index.js
│   ├── about.js
│   └── blog/
│       ├── [slug].js
│       └── index.js
├── styles/
│   ├── globals.css
│   └── utils.css
├── utils/
│   └── api.js
└── next.config.js
```

**other reference:** [fe.engineer: Scaling Application](https://www.fe.engineer/handbook/scaling-applications)

**Components**:

```jsx
// components/Header/Header.js
import styles from "./Header.module.css";

const Header = () => {
    return (
        <header className={styles.header}>
            <nav>{/* Navigation links */}</nav>
        </header>
    );
};

export default Header;
```

`Spreading props with Native HTML Attribute`

```tsx
// Dirty code
interface InputGroupProps {
    label: string;
    type?: string;
    required?: boolean;
    placeholder?: string;
    id?: string;
}
function InputGroup({
    label,
    type,
    required,
    placeholder,
    id,
}: InputGroupProps) {
    return (
        <label>
            {label}:
            <input
                type={type}
                required={required}
                placeholder={placeholder}
                id={id}
            />
        </label>
    );
}
```

```tsx
// Clean code
import type { ComponentPropsWithoutRef } from "react";

interface InputGroupProps extends ComponentPropsWithoutRef<"input"> {
    label: string; // only add "special" props
}

function InputGroup({
    label,
    type,
    required,
    placeholder,
    id,
}: InputGroupProps) {
    return (
        <label>
            {label}:
            <input
                type={type}
                required={required}
                placeholder={placeholder}
                id={id}
            />
        </label>
    );
}
```

**State Management**:
Gunakan state management untuk menghindari [prop drilling](https://arc.net/l/quote/ajhdejda)

```jsx
import { useAtom } from 'jotai'

const AnimeApp = () => {
  const [anime, setAnime] = useAtom(animeAtom)

  return (
    <>
      <ul>
        {anime.map((item) => (
          <li key={item.title}>{item.title}</li>
        ))}
      </ul>
      <button onClick={() => {
        setAnime((anime) => [
          ...anime,
          {
            title: 'Cowboy Bebop',
            year: 1998,
            watched: false
          }
        ])
      }}>
        Add Cowboy Bebop
      </button>
    <>
  )
}
```

Read more: [State Management](https://arc.net/l/quote/ybzmhcka)

**Linting and Formatting**:

`.eslintrc.js`:

```js
module.exports = {
    extends: ["next/core-web-vitals", "prettier"],
    rules: {
        // Custom ESLint rules
        "react/prop-types": "off",
        "react/react-in-jsx-scope": "off",
    },
};
```

`.prettierrc.js`:

```js
module.exports = {
    semi: true,
    trailingComma: "all",
    singleQuote: true,
    printWidth: 80,
    tabWidth: 2,
};
```

`lint staged`

```bash
$ git commit

✔ Preparing lint-staged...
❯ Running tasks for staged files...
❯ packages/frontend/.lintstagedrc.json — 1 file
↓ _.js — no files [SKIPPED]
❯ _.{json,md} — 1 file
⠹ prettier --write
↓ packages/backend/.lintstagedrc.json — 2 files
❯ _.js — 2 files
⠼ eslint --fix
↓ _.{json,md} — no files [SKIPPED]
◼ Applying modifications from tasks...
◼ Cleaning up temporary files...
```

`husky`: https://typicode.github.io/husky/get-started.html

**Performance Optimization**:

```jsx
// pages/blog/index.js
import Link from "next/link";

const BlogList = ({ posts }) => {
    return (
        <div>
            <h1>Blog Posts</h1>
            <ul>
                {posts.map((post) => (
                    <li key={post.slug}>
                        <Link href={`/blog/${post.slug}`}>
                            <a>{post.title}</a>
                        </Link>
                    </li>
                ))}
            </ul>
        </div>
    );
};

export const getStaticProps = async () => {
    const res = await fetch(`https://api.example.com/blog`);
    const posts = await res.json();

    return {
        props: {
            posts,
        },
        revalidate: 60, // Revalidate the cache every 60 seconds
    };
};

export default BlogList;
```

**Documentation**:

```jsx
/**
 * Header component
 *
 * @module components/Header
 */

import styles from "./Header.module.css";

/**
 * Header component that displays navigation links
 *
 * @component
 * @example
 * return (
 *   <Header />
 * )
 */
const Header = () => {
    return (
        <header className={styles.header}>
            <nav>{/* Navigation links */}</nav>
        </header>
    );
};

export default Header;
```

**Data Fetching**

> Separate logic from UI by creating custom hooks

```jsx
export const Products = () => {
    const { products, isLoading, error } = useProducts();

    if (isLoading) {
        return <div>Loading...</div>;
    }

    if (error) {
        return <div>Error: {error.message}</div>;
    }

    return (
        <ul>
            {products.map((product) => (
                <li key={product.id}>{product.title}</li>
            ))}
        </ul>
    );
};
```

Recommended libraries For fetching data

`Axios`: a promise-based HTTP client for the browser and Node.js. It provides an easy-to-use API for making HTTP requests.
For linking to state

`React Query`: a data fetching library that provides a simple and concise API for fetching, caching, synchronizing, and updating server state in React.

`SWR (Stale-While-Revalidate)`: a lightweight library that provides simple and efficient data fetching and caching, with automatic revalidation and stale data usage.

**DRY (Do Not Repeat Yourself) Principle**  
Code principle guna mengurangi redudansi / code reusability

`Reusable Component`

```jsx
import type { FC, ComponentPropsWithoutRef } from "react";

interface Props extends ComponentPropsWithoutRef<"button"> {
    title: string;
}

const Button: FC<Props> = ({ title, ...props }) => {
    return <button {...props}>{title}</button>;
};
```

`Custom Hooks`
```js
// hooks/use-form.js
import { useState } from 'react';

const useForm = (initialValues) => {
  const [values, setValues] = useState(initialValues);

  const handleChange = (event) => {
    setValues({
      ...values,
      [event.target.name]: event.target.value,
    });
  };

  const resetForm = () => {
    setValues(initialValues);
  };

  return { values, handleChange, resetForm };
};

export default useForm;

------------------------------------------------------------

// components/ContactForm.js
import useForm from '../hooks/useForm';

const ContactForm = () => {
  const { values, handleChange, resetForm } = useForm({
    name: '',
    email: '',
    message: '',
  });

  const handleSubmit = (event) => {
    event.preventDefault();
    // Submit form data
    resetForm();
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
};

export default ContactForm;
```

`Utility function`
```js
const formatDate = (dateString) => {
  const date = new Date(dateString);
  return date.toLocaleDateString();
};

export default formatDate;
```

**KISS Principle**  
_KISS (Keep It Simple, Stupid) principle is a software design principle that emphasizes simplicity and clarity over complexity. In Next.js and React applications, there are several ways to apply the KISS principle_

`Hindari premature optimization`
- Fokus pada penulisan kode yang sederhana dan mudah dibaca terlebih dahulu, sebelum mengoptimalkan performa.
- Premature optimization dapat menyebabkan kode menjadi terlalu rumit dan sulit dimaintain.
- Identifikasi dan optimalkan hambatan performa hanya jika diperlukan.