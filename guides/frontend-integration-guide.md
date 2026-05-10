# Frontend Integration Guide

## Configuration

```ts
const API = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000";
```

## Authentification

```ts
export async function login(email: string, password: string) {
  const res = await fetch(`${API}/api/v1/auth/login`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password }),
  });
  return res.json();
}
```

## Jobs

```ts
export async function getJobs(token: string) {
  const res = await fetch(`${API}/api/v1/jobs`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  return res.json();
}
```

## WebSocket progression

```ts
const ws = new WebSocket(
  `ws://localhost:8000/api/v1/jobs/ws/${jobId}/progress?token=${token}`,
);

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(data.progress, data.status);
};
```

## Modules frontend v3.5

Le frontend public ou prive peut exposer:

- Dashboard campagnes.
- Card Studio no-code.
- Jobs Studio.
- Jobs DHIS2.
- Portail verification.
- Facturation et SmartUsageCard.
- Administration programmes et plans.

## Integrations

Les integrations DHIS2, WhatsApp, SMS et Google Wallet doivent etre configurees cote backend. Le frontend ne doit jamais manipuler les secrets fournisseurs.
