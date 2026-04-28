# Jofarm Business Management System

A secure, role-based business management system built with Flask, deployed via GitHub and Cloudflare. It manages stock flow, client records, and employee dashboards with strict access control. Clients can perform online financial transactions, with live location, time, and date captured for auditing.

---

## 🚀 Features
- Secure login with Bcrypt password hashing
- Role-based dashboards (CEO, Manager, Subordinate)
- Company-only registration (@jofarm.com emails)
- Client-side financial transactions with audit trail
- Location, time, and date capture for compliance
- Cloudflare proxy with HTTPS and DDoS protection
- Strong security headers and hidden file protection

---

## 📂 Structure

---

## ⚙️ Deployment
1. Push code to GitHub.
2. Cloudflare pulls and deploys automatically.
3. Flask app runs behind Cloudflare proxy.
4. Role-based dashboards restrict access by employee level.
5. Clients transact securely with audit logging.

---

## 🔒 Security
- Passwords hashed with Bcrypt
- Role-based access enforced in Flask
- Sensitive files blocked by Apache
- Cloudflare firewall + HTTPS redirect
- Transactions logged with location, time, and date
