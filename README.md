# Deadstock Inventory Management System

A full-stack inventory management system built for the **fast-fashion supply chain**, designed to track deadstock items from branch intake through warehouse consolidation to final sustainable disposition (recycle, donate, resell, upcycle, rebrand, or dispose).

Built as a DBMS mini-project with a focus on real-world role-based workflows, automated notifications, and sustainability tracking.

---

## Features

### Core Inventory Management
- **4-role RBAC system** — Admin, Branch Head, Warehouse Head, Stock Allocation Head, each with scoped permissions enforced at the route level
- **SKU-based item tracking** with auto-generated unique SKUs per item
- **Multi-stage deadstock lifecycle** — Branch → Warehouse → Stock Allocation → Final Disposition
- **Bulk send-to-warehouse** and **bulk allocation** workflows
- **Safe deletion flows** — warehouse deletion requires branch reassignment; soft-delete archiving for branches and warehouses

### Authentication & Notifications
- Role-based login with session management
- **OTP-based password reset** (6-digit, auto-advance UI) delivered via **Gmail SMTP** and **Twilio WhatsApp** simultaneously
- **Automated daily aging alerts** via APScheduler — flags items unsent to warehouse for 30+ days
- WhatsApp alerts to warehouse heads when branches dispatch stock

### Reporting & Analytics
- **PDF branch reports** generated with ReportLab, downloadable or emailed via **SendGrid**
- **Chart.js dashboards** — bar charts, pie charts, and line trend charts for allocation data
- **Sustainability scorecard** — star-rated branch ratings, auto-updated via a MySQL trigger on every stock allocation
- **Waste reduction tracking** (kg) and historical rating trends per branch
- Live table search, column sorting, and dark mode across all data views

---

## Tech Stack

| Layer          | Technology                          |
|----------------|--------------------------------------|
| Backend        | Python, Flask                        |
| Database       | MySQL (11 tables, 3NF normalized)    |
| Frontend       | HTML, CSS, JavaScript, Chart.js      |
| PDF Generation | ReportLab                            |
| Email          | Gmail SMTP, SendGrid                 |
| WhatsApp       | Twilio API                           |
| Scheduling     | APScheduler                          |

---

## Database Design

- **11 core tables**: `ADMIN`, `HEAD`, `CONTACTS`, `MATERIAL`, `WAREHOUSE`, `BRANCH`, `DEADSTOCK`, `STOCK_ALLOCATION`, `REPORT`, `RECYCLE_RECORD`, `RATING_HISTORY`
- Normalized to **3NF** with foreign key constraints and `ON DELETE RESTRICT`/`CASCADE` rules to preserve data integrity
- **MySQL triggers**:
  - `trg_update_branch_rating` — auto-adjusts branch sustainability rating on every allocation
  - `trg_rating_history` — logs rating changes over time
  - `trg_insert_recycle_record` — auto-creates a recycle audit record on recycle allocations
- Soft-delete archive tables (`DELETED_BRANCH`, `DELETED_WAREHOUSE`) to preserve historical data

---

## User Roles

| Role               | Permissions                                                   |
|--------------------|----------------------------------------------------------------|
| **Admin**          | Full access — manage branches, warehouses, heads, materials, view all reports |
| **Branch Head**     | Add deadstock, send to warehouse, view own branch report      |
| **Warehouse Head**  | Receive deadstock from branches, forward to Stock Allocation  |
| **Stock Allocation Head** | Allocate items to final disposition, view/download/email branch reports |

---

## Setup

### Prerequisites
- Python 3.8+
- MySQL Server
- pip

### Installation

```bash
# Clone the repository
git clone <your-repo-url>
cd deadstock-ims

# Install dependencies
pip install flask mysql-connector-python reportlab twilio apscheduler sendgrid python-dotenv

# Set up the database
mysql -u root -p < database_queries.txt

# Configure environment variables (see Configuration below)
# Run the app
python app.py
```

The app will be available at `http://localhost:5000`

### Configuration

Create a `.env` file in the project root (do **not** commit this file):

```env
MYSQL_HOST=127.0.0.1
MYSQL_USER=root
MYSQL_PASSWORD=your_mysql_password
MYSQL_DATABASE=deadstock_db

MAIL_SENDER=your_email@gmail.com
MAIL_PASSWORD=your_gmail_app_password

TWILIO_SID=your_twilio_sid
TWILIO_TOKEN=your_twilio_token
TWILIO_FROM=whatsapp:+14155238886

SENDGRID_API_KEY=your_sendgrid_api_key
SENDGRID_FROM=noreply@yourdomain.com
```

 **Note:** Update `app.py` to load these values via `python-dotenv` (`os.environ.get(...)`) instead of hardcoded credentials before deploying.

---

##  Project Structure

```
deadstock-ims/
├── app.py                  # Main Flask application
├── database_queries.txt    # Full DB schema, triggers, and migrations
├── templates/
│   ├── landing.html
│   ├── login.html
│   ├── dashboard.html
│   ├── add_*.html          # Add Branch / Warehouse / Head / Material / Deadstock / Contacts
│   ├── show_*.html         # Table views with search/sort
│   ├── sustainability.html # Scorecard with Chart.js trends
│   ├── report.html         # Branch-level report
│   ├── sa_branch_report.html
│   ├── admin_branch_report.html
│   ├── verify_otp.html     # OTP entry UI
│   ├── reset_password.html
│   ├── reassign_warehouse.html
│   └── 404.html / 500.html
└── README.md
```

---

## Sustainability Impact

This system was designed with sustainability as a core metric, not an afterthought:

- Tracks **waste reduced (kg)** across every branch
- Calculates a **green ratio** (recycled + donated + upcycled vs. disposed) per branch
- Branch ratings (0–5 stars) automatically reward sustainable disposition choices and penalize disposal
- Historical rating trends visualized per branch over time

---

## Notes

- This project was built as an academic DBMS mini-project and demo system
- Credentials in the codebase should be moved to environment variables before any production use
- WhatsApp alerts use the Twilio sandbox; production use requires a verified Twilio number

---

## License

This project is for academic and portfolio purposes.
