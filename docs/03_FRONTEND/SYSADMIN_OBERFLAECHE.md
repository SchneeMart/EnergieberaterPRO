# Sysadmin-Oberfläche

## Übersicht

Die Sysadmin-Oberfläche bietet vollständige Kontrolle über die gesamte EnergieberaterPRO-Plattform. Der Sysadmin kann ALLES einstellen - von Organisationsverwaltung über Systemkonfiguration bis zur Feature-Freischaltung.

## Architektur

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SYSADMIN DASHBOARD                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Organisationen│  │    Benutzer  │  │   Abrechnung │  │    System    │    │
│  │  verwalten   │  │   verwalten  │  │   verwalten  │  │    config    │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │    Module    │  │   Templates  │  │     Logs &   │  │   Support &  │    │
│  │  & Features  │  │  verwalten   │  │    Audits    │  │   Tickets    │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Haupt-Navigation

```html
<!-- /templates/sysadmin/layout.html -->
<div class="sysadmin-layout" x-data="sysadminLayout()">
    <!-- Sidebar Navigation -->
    <aside class="sysadmin-sidebar">
        <div class="sidebar-header">
            <img src="/static/img/logo-admin.svg" alt="EnergieberaterPRO Admin" class="logo">
            <span class="badge badge-danger">SYSADMIN</span>
        </div>

        <nav class="sidebar-nav">
            <!-- Dashboard -->
            <a href="/sysadmin/dashboard" class="nav-item"
               :class="{ 'active': currentPage === 'dashboard' }">
                <svg class="icon"><use href="#icon-dashboard"/></svg>
                <span>Dashboard</span>
            </a>

            <!-- Organisationen -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('orgs')">
                    <svg class="icon"><use href="#icon-building"/></svg>
                    <span>Organisationen</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.orgs }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.orgs">
                    <a href="/sysadmin/organizations">Alle Organisationen</a>
                    <a href="/sysadmin/organizations/new">Neue Organisation</a>
                    <a href="/sysadmin/organizations/pending">Ausstehende Anträge</a>
                    <a href="/sysadmin/organizations/suspended">Gesperrte</a>
                </div>
            </div>

            <!-- Benutzer -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('users')">
                    <svg class="icon"><use href="#icon-users"/></svg>
                    <span>Benutzer</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.users }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.users">
                    <a href="/sysadmin/users">Alle Benutzer</a>
                    <a href="/sysadmin/users/admins">Org-Admins</a>
                    <a href="/sysadmin/users/sessions">Aktive Sessions</a>
                    <a href="/sysadmin/users/impersonate">Support-Zugang</a>
                </div>
            </div>

            <!-- Kunden & Projekte (Cross-Org) -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('business')">
                    <svg class="icon"><use href="#icon-briefcase"/></svg>
                    <span>Geschäftsdaten</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.business }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.business">
                    <a href="/sysadmin/customers">Alle Kunden</a>
                    <a href="/sysadmin/projects">Alle Projekte</a>
                    <a href="/sysadmin/orders">Alle Aufträge</a>
                    <a href="/sysadmin/invoices">Alle Rechnungen</a>
                </div>
            </div>

            <!-- Abrechnung -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('billing')">
                    <svg class="icon"><use href="#icon-credit-card"/></svg>
                    <span>Abrechnung</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.billing }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.billing">
                    <a href="/sysadmin/billing/overview">Übersicht</a>
                    <a href="/sysadmin/billing/subscriptions">Abonnements</a>
                    <a href="/sysadmin/billing/invoices">Plattform-Rechnungen</a>
                    <a href="/sysadmin/billing/pricing">Preisgestaltung</a>
                    <a href="/sysadmin/billing/coupons">Gutscheine</a>
                </div>
            </div>

            <!-- Module & Features -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('modules')">
                    <svg class="icon"><use href="#icon-puzzle"/></svg>
                    <span>Module</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.modules }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.modules">
                    <a href="/sysadmin/modules">Module verwalten</a>
                    <a href="/sysadmin/modules/features">Feature-Flags</a>
                    <a href="/sysadmin/modules/packages">Pakete</a>
                    <a href="/sysadmin/modules/limits">Limits</a>
                </div>
            </div>

            <!-- Templates -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('templates')">
                    <svg class="icon"><use href="#icon-template"/></svg>
                    <span>Templates</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.templates }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.templates">
                    <a href="/sysadmin/templates/reports">Berichts-Templates</a>
                    <a href="/sysadmin/templates/emails">E-Mail-Templates</a>
                    <a href="/sysadmin/templates/documents">Dokument-Templates</a>
                    <a href="/sysadmin/templates/projects">Projekt-Templates</a>
                </div>
            </div>

            <!-- System -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('system')">
                    <svg class="icon"><use href="#icon-cog"/></svg>
                    <span>System</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.system }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.system">
                    <a href="/sysadmin/system/config">Konfiguration</a>
                    <a href="/sysadmin/system/security">Sicherheit</a>
                    <a href="/sysadmin/system/email">E-Mail-Server</a>
                    <a href="/sysadmin/system/storage">Speicher</a>
                    <a href="/sysadmin/system/ai">KI-Einstellungen</a>
                    <a href="/sysadmin/system/integrations">Integrationen</a>
                    <a href="/sysadmin/system/maintenance">Wartung</a>
                </div>
            </div>

            <!-- Logs & Audits -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('logs')">
                    <svg class="icon"><use href="#icon-file-text"/></svg>
                    <span>Logs & Audits</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.logs }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.logs">
                    <a href="/sysadmin/logs/audit">Audit-Logs</a>
                    <a href="/sysadmin/logs/security">Sicherheits-Logs</a>
                    <a href="/sysadmin/logs/errors">Fehler-Logs</a>
                    <a href="/sysadmin/logs/api">API-Logs</a>
                    <a href="/sysadmin/logs/analytics">Analytics</a>
                </div>
            </div>

            <!-- Support -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('support')">
                    <svg class="icon"><use href="#icon-headset"/></svg>
                    <span>Support</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.support }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.support">
                    <a href="/sysadmin/support/tickets">Tickets</a>
                    <a href="/sysadmin/support/announcements">Ankündigungen</a>
                    <a href="/sysadmin/support/knowledge">Wissensdatenbank</a>
                </div>
            </div>
        </nav>
    </aside>

    <!-- Main Content -->
    <main class="sysadmin-main">
        <header class="sysadmin-header">
            <div class="header-left">
                <button @click="toggleSidebar" class="btn-icon">
                    <svg class="icon"><use href="#icon-menu"/></svg>
                </button>
                <div class="breadcrumbs" x-html="breadcrumbs"></div>
            </div>
            <div class="header-right">
                <div class="search-global">
                    <input type="search" placeholder="Suche..." @keyup.enter="globalSearch">
                    <kbd>⌘K</kbd>
                </div>
                <button class="btn-icon" @click="openNotifications">
                    <svg class="icon"><use href="#icon-bell"/></svg>
                    <span class="badge" x-show="notifications > 0" x-text="notifications"></span>
                </button>
                <div class="user-menu" @click.away="userMenuOpen = false">
                    <button @click="userMenuOpen = !userMenuOpen">
                        <img :src="user.avatar" alt="" class="avatar">
                        <span x-text="user.name"></span>
                    </button>
                    <div class="dropdown" x-show="userMenuOpen">
                        <a href="/sysadmin/profile">Profil</a>
                        <a href="/sysadmin/settings">Einstellungen</a>
                        <hr>
                        <a href="/logout" class="text-danger">Abmelden</a>
                    </div>
                </div>
            </div>
        </header>

        <div class="sysadmin-content">
            <!-- Page content injected here -->
        </div>
    </main>
</div>
```

## Dashboard-Übersicht

```html
<!-- /templates/sysadmin/dashboard.html -->
<div class="dashboard" x-data="sysadminDashboard()">
    <!-- Key Metrics -->
    <section class="metrics-grid">
        <div class="metric-card">
            <div class="metric-icon bg-blue">
                <svg><use href="#icon-building"/></svg>
            </div>
            <div class="metric-content">
                <span class="metric-label">Organisationen</span>
                <span class="metric-value" x-text="metrics.organizations.total">0</span>
                <span class="metric-change positive" x-show="metrics.organizations.change > 0">
                    +<span x-text="metrics.organizations.change"></span> diese Woche
                </span>
            </div>
        </div>

        <div class="metric-card">
            <div class="metric-icon bg-green">
                <svg><use href="#icon-users"/></svg>
            </div>
            <div class="metric-content">
                <span class="metric-label">Aktive Benutzer</span>
                <span class="metric-value" x-text="metrics.users.active">0</span>
                <span class="metric-sub" x-text="`von ${metrics.users.total} gesamt`"></span>
            </div>
        </div>

        <div class="metric-card">
            <div class="metric-icon bg-purple">
                <svg><use href="#icon-folder"/></svg>
            </div>
            <div class="metric-content">
                <span class="metric-label">Projekte (gesamt)</span>
                <span class="metric-value" x-text="metrics.projects.total">0</span>
                <span class="metric-sub" x-text="`${metrics.projects.active} aktiv`"></span>
            </div>
        </div>

        <div class="metric-card">
            <div class="metric-icon bg-yellow">
                <svg><use href="#icon-euro"/></svg>
            </div>
            <div class="metric-content">
                <span class="metric-label">Monatsumsatz</span>
                <span class="metric-value" x-text="formatCurrency(metrics.revenue.monthly)">€0</span>
                <span class="metric-change" :class="metrics.revenue.change >= 0 ? 'positive' : 'negative'">
                    <span x-text="metrics.revenue.change >= 0 ? '+' : ''"></span>
                    <span x-text="metrics.revenue.change"></span>% zum Vormonat
                </span>
            </div>
        </div>
    </section>

    <!-- Secondary Metrics -->
    <section class="metrics-secondary">
        <div class="metric-small">
            <span class="label">Offene Tickets</span>
            <span class="value" :class="{ 'text-danger': metrics.tickets.open > 10 }"
                  x-text="metrics.tickets.open">0</span>
        </div>
        <div class="metric-small">
            <span class="label">Ausstehende Anträge</span>
            <span class="value" :class="{ 'text-warning': metrics.pending.organizations > 0 }"
                  x-text="metrics.pending.organizations">0</span>
        </div>
        <div class="metric-small">
            <span class="label">Fehlgeschlagene Jobs</span>
            <span class="value" :class="{ 'text-danger': metrics.jobs.failed > 0 }"
                  x-text="metrics.jobs.failed">0</span>
        </div>
        <div class="metric-small">
            <span class="label">Speichernutzung</span>
            <span class="value" x-text="`${metrics.storage.used}%`">0%</span>
        </div>
        <div class="metric-small">
            <span class="label">API-Anfragen/Min</span>
            <span class="value" x-text="metrics.api.rpm">0</span>
        </div>
    </section>

    <!-- Charts Row -->
    <section class="charts-row">
        <div class="chart-card">
            <h3>Benutzer-Aktivität</h3>
            <canvas id="userActivityChart"></canvas>
        </div>
        <div class="chart-card">
            <h3>Umsatz nach Modul</h3>
            <canvas id="revenueByModuleChart"></canvas>
        </div>
    </section>

    <!-- Activity & Alerts -->
    <section class="activity-alerts">
        <div class="activity-feed">
            <h3>Letzte Aktivitäten</h3>
            <ul class="activity-list">
                <template x-for="activity in recentActivity" :key="activity.id">
                    <li class="activity-item">
                        <div class="activity-icon" :class="`bg-${activity.color}`">
                            <svg><use :href="`#icon-${activity.icon}`"/></svg>
                        </div>
                        <div class="activity-content">
                            <p x-html="activity.message"></p>
                            <time x-text="formatTime(activity.timestamp)"></time>
                        </div>
                    </li>
                </template>
            </ul>
            <a href="/sysadmin/logs/audit" class="btn btn-link">Alle Aktivitäten</a>
        </div>

        <div class="alerts-panel">
            <h3>System-Warnungen</h3>
            <div class="alerts-list">
                <template x-for="alert in systemAlerts" :key="alert.id">
                    <div class="alert-item" :class="`alert-${alert.severity}`">
                        <div class="alert-header">
                            <span class="alert-title" x-text="alert.title"></span>
                            <button @click="dismissAlert(alert.id)" class="btn-icon-sm">×</button>
                        </div>
                        <p class="alert-message" x-text="alert.message"></p>
                        <time x-text="formatTime(alert.timestamp)"></time>
                    </div>
                </template>
            </div>
        </div>
    </section>
</div>
```

## Organisationsverwaltung

### Organisations-Liste

```html
<!-- /templates/sysadmin/organizations/list.html -->
<div class="page-organizations" x-data="organizationsPage()">
    <header class="page-header">
        <h1>Organisationen</h1>
        <div class="header-actions">
            <button @click="exportOrganizations" class="btn btn-secondary">
                <svg class="icon"><use href="#icon-download"/></svg>
                Export
            </button>
            <a href="/sysadmin/organizations/new" class="btn btn-primary">
                <svg class="icon"><use href="#icon-plus"/></svg>
                Neue Organisation
            </a>
        </div>
    </header>

    <!-- Filters -->
    <div class="filters-bar">
        <div class="search-input">
            <svg class="icon"><use href="#icon-search"/></svg>
            <input type="search" placeholder="Organisation suchen..."
                   x-model="filters.search" @input.debounce.300ms="applyFilters">
        </div>

        <select x-model="filters.status" @change="applyFilters">
            <option value="">Alle Status</option>
            <option value="active">Aktiv</option>
            <option value="trial">Test</option>
            <option value="suspended">Gesperrt</option>
            <option value="pending">Ausstehend</option>
        </select>

        <select x-model="filters.plan" @change="applyFilters">
            <option value="">Alle Pläne</option>
            <option value="starter">Starter</option>
            <option value="professional">Professional</option>
            <option value="enterprise">Enterprise</option>
            <option value="custom">Custom</option>
        </select>

        <select x-model="filters.sortBy" @change="applyFilters">
            <option value="created_desc">Neueste zuerst</option>
            <option value="created_asc">Älteste zuerst</option>
            <option value="name_asc">Name A-Z</option>
            <option value="name_desc">Name Z-A</option>
            <option value="users_desc">Meiste Benutzer</option>
            <option value="revenue_desc">Höchster Umsatz</option>
        </select>
    </div>

    <!-- Organizations Table -->
    <div class="data-table-container">
        <table class="data-table">
            <thead>
                <tr>
                    <th>
                        <input type="checkbox" @change="toggleSelectAll"
                               :checked="selectedAll" :indeterminate="selectedSome">
                    </th>
                    <th>Organisation</th>
                    <th>Status</th>
                    <th>Plan</th>
                    <th>Benutzer</th>
                    <th>Projekte</th>
                    <th>Monatl. Umsatz</th>
                    <th>Erstellt</th>
                    <th>Aktionen</th>
                </tr>
            </thead>
            <tbody>
                <template x-for="org in organizations" :key="org.id">
                    <tr :class="{ 'row-suspended': org.status === 'suspended' }">
                        <td>
                            <input type="checkbox" :value="org.id" x-model="selected">
                        </td>
                        <td>
                            <div class="org-cell">
                                <img :src="org.logo || '/static/img/org-default.svg'"
                                     alt="" class="org-logo">
                                <div>
                                    <a :href="`/sysadmin/organizations/${org.id}`"
                                       class="org-name" x-text="org.name"></a>
                                    <span class="org-slug" x-text="org.slug"></span>
                                </div>
                            </div>
                        </td>
                        <td>
                            <span class="badge" :class="`badge-${getStatusColor(org.status)}`"
                                  x-text="getStatusLabel(org.status)"></span>
                        </td>
                        <td x-text="org.plan"></td>
                        <td>
                            <span x-text="org.users_count"></span>
                            <span class="text-muted" x-text="`/ ${org.users_limit || '∞'}`"></span>
                        </td>
                        <td x-text="org.projects_count"></td>
                        <td x-text="formatCurrency(org.monthly_revenue)"></td>
                        <td x-text="formatDate(org.created_at)"></td>
                        <td>
                            <div class="actions-menu" @click.away="openMenu = null">
                                <button @click="openMenu = org.id" class="btn-icon">
                                    <svg><use href="#icon-dots-vertical"/></svg>
                                </button>
                                <div class="dropdown" x-show="openMenu === org.id">
                                    <a :href="`/sysadmin/organizations/${org.id}`">Details</a>
                                    <a :href="`/sysadmin/organizations/${org.id}/edit`">Bearbeiten</a>
                                    <a :href="`/sysadmin/organizations/${org.id}/users`">Benutzer</a>
                                    <a :href="`/sysadmin/organizations/${org.id}/billing`">Abrechnung</a>
                                    <hr>
                                    <button @click="impersonateAdmin(org.id)">
                                        Als Admin anmelden
                                    </button>
                                    <hr>
                                    <template x-if="org.status !== 'suspended'">
                                        <button @click="suspendOrganization(org.id)" class="text-danger">
                                            Sperren
                                        </button>
                                    </template>
                                    <template x-if="org.status === 'suspended'">
                                        <button @click="activateOrganization(org.id)" class="text-success">
                                            Aktivieren
                                        </button>
                                    </template>
                                </div>
                            </div>
                        </td>
                    </tr>
                </template>
            </tbody>
        </table>
    </div>

    <!-- Bulk Actions -->
    <div class="bulk-actions" x-show="selected.length > 0">
        <span x-text="`${selected.length} ausgewählt`"></span>
        <button @click="bulkExport" class="btn btn-sm">Export</button>
        <button @click="bulkSendEmail" class="btn btn-sm">E-Mail senden</button>
        <button @click="bulkSuspend" class="btn btn-sm btn-danger">Sperren</button>
    </div>

    <!-- Pagination -->
    <div class="pagination">
        <span x-text="`${pagination.from}-${pagination.to} von ${pagination.total}`"></span>
        <div class="pagination-controls">
            <button @click="prevPage" :disabled="pagination.page === 1">←</button>
            <template x-for="page in paginationPages" :key="page">
                <button @click="goToPage(page)"
                        :class="{ 'active': page === pagination.page }"
                        x-text="page"></button>
            </template>
            <button @click="nextPage" :disabled="pagination.page === pagination.lastPage">→</button>
        </div>
        <select x-model="pagination.perPage" @change="applyFilters">
            <option value="25">25 pro Seite</option>
            <option value="50">50 pro Seite</option>
            <option value="100">100 pro Seite</option>
        </select>
    </div>
</div>
```

### Organisations-Detail

```html
<!-- /templates/sysadmin/organizations/detail.html -->
<div class="page-org-detail" x-data="orgDetailPage()">
    <header class="page-header">
        <div class="header-left">
            <a href="/sysadmin/organizations" class="back-link">← Zurück</a>
            <div class="org-header">
                <img :src="organization.logo" alt="" class="org-logo-lg">
                <div>
                    <h1 x-text="organization.name"></h1>
                    <span class="org-slug" x-text="organization.slug"></span>
                    <span class="badge" :class="`badge-${getStatusColor(organization.status)}`"
                          x-text="getStatusLabel(organization.status)"></span>
                </div>
            </div>
        </div>
        <div class="header-actions">
            <button @click="impersonateAdmin" class="btn btn-secondary">
                <svg class="icon"><use href="#icon-user-switch"/></svg>
                Als Admin anmelden
            </button>
            <a :href="`/sysadmin/organizations/${organization.id}/edit`" class="btn btn-primary">
                Bearbeiten
            </a>
        </div>
    </header>

    <!-- Tabs -->
    <nav class="tabs">
        <button @click="activeTab = 'overview'"
                :class="{ 'active': activeTab === 'overview' }">Übersicht</button>
        <button @click="activeTab = 'users'"
                :class="{ 'active': activeTab === 'users' }">Benutzer</button>
        <button @click="activeTab = 'customers'"
                :class="{ 'active': activeTab === 'customers' }">Kunden</button>
        <button @click="activeTab = 'projects'"
                :class="{ 'active': activeTab === 'projects' }">Projekte</button>
        <button @click="activeTab = 'billing'"
                :class="{ 'active': activeTab === 'billing' }">Abrechnung</button>
        <button @click="activeTab = 'modules'"
                :class="{ 'active': activeTab === 'modules' }">Module</button>
        <button @click="activeTab = 'settings'"
                :class="{ 'active': activeTab === 'settings' }">Einstellungen</button>
        <button @click="activeTab = 'logs'"
                :class="{ 'active': activeTab === 'logs' }">Logs</button>
    </nav>

    <!-- Tab Content: Overview -->
    <div class="tab-content" x-show="activeTab === 'overview'">
        <div class="overview-grid">
            <!-- Stats Cards -->
            <div class="stats-cards">
                <div class="stat-card">
                    <span class="stat-value" x-text="organization.users_count"></span>
                    <span class="stat-label">Benutzer</span>
                </div>
                <div class="stat-card">
                    <span class="stat-value" x-text="organization.customers_count"></span>
                    <span class="stat-label">Kunden</span>
                </div>
                <div class="stat-card">
                    <span class="stat-value" x-text="organization.projects_count"></span>
                    <span class="stat-label">Projekte</span>
                </div>
                <div class="stat-card">
                    <span class="stat-value" x-text="formatCurrency(organization.monthly_revenue)"></span>
                    <span class="stat-label">Monatl. Umsatz</span>
                </div>
            </div>

            <!-- Info Card -->
            <div class="info-card">
                <h3>Organisation</h3>
                <dl class="info-list">
                    <dt>ID</dt>
                    <dd x-text="organization.id"></dd>

                    <dt>Erstellt</dt>
                    <dd x-text="formatDateTime(organization.created_at)"></dd>

                    <dt>Plan</dt>
                    <dd x-text="organization.plan"></dd>

                    <dt>Abrechnungszyklus</dt>
                    <dd x-text="organization.billing_cycle"></dd>

                    <dt>Nächste Rechnung</dt>
                    <dd x-text="formatDate(organization.next_invoice_date)"></dd>

                    <dt>Speichernutzung</dt>
                    <dd x-text="`${organization.storage_used} / ${organization.storage_limit}`"></dd>
                </dl>
            </div>

            <!-- Contact Card -->
            <div class="info-card">
                <h3>Kontakt</h3>
                <dl class="info-list">
                    <dt>Primärer Admin</dt>
                    <dd>
                        <a :href="`/sysadmin/users/${organization.primary_admin.id}`"
                           x-text="organization.primary_admin.name"></a>
                    </dd>

                    <dt>E-Mail</dt>
                    <dd x-text="organization.email"></dd>

                    <dt>Telefon</dt>
                    <dd x-text="organization.phone || '-'"></dd>

                    <dt>Adresse</dt>
                    <dd x-html="formatAddress(organization.address)"></dd>

                    <dt>Website</dt>
                    <dd>
                        <a :href="organization.website" target="_blank"
                           x-text="organization.website || '-'"></a>
                    </dd>
                </dl>
            </div>

            <!-- Recent Activity -->
            <div class="activity-card">
                <h3>Letzte Aktivität</h3>
                <ul class="activity-list-sm">
                    <template x-for="activity in organization.recent_activity" :key="activity.id">
                        <li>
                            <span x-text="activity.user_name"></span>
                            <span x-text="activity.action"></span>
                            <time x-text="formatTimeAgo(activity.timestamp)"></time>
                        </li>
                    </template>
                </ul>
            </div>
        </div>
    </div>

    <!-- Tab Content: Users -->
    <div class="tab-content" x-show="activeTab === 'users'">
        <div class="table-actions">
            <input type="search" placeholder="Benutzer suchen..."
                   x-model="usersFilter" @input.debounce.300ms="filterUsers">
            <button @click="openAddUserModal" class="btn btn-primary">
                + Benutzer hinzufügen
            </button>
        </div>

        <table class="data-table">
            <thead>
                <tr>
                    <th>Name</th>
                    <th>E-Mail</th>
                    <th>Rolle</th>
                    <th>Status</th>
                    <th>Letzter Login</th>
                    <th>Aktionen</th>
                </tr>
            </thead>
            <tbody>
                <template x-for="user in filteredUsers" :key="user.id">
                    <tr>
                        <td>
                            <div class="user-cell">
                                <img :src="user.avatar" alt="" class="avatar-sm">
                                <span x-text="user.name"></span>
                            </div>
                        </td>
                        <td x-text="user.email"></td>
                        <td>
                            <span class="badge" :class="getRoleBadge(user.role)"
                                  x-text="user.role"></span>
                        </td>
                        <td>
                            <span class="status-dot" :class="user.active ? 'active' : 'inactive'"></span>
                            <span x-text="user.active ? 'Aktiv' : 'Inaktiv'"></span>
                        </td>
                        <td x-text="formatDateTime(user.last_login)"></td>
                        <td>
                            <button @click="impersonateUser(user.id)" class="btn btn-sm">
                                Impersonate
                            </button>
                            <button @click="editUser(user.id)" class="btn btn-sm">
                                Bearbeiten
                            </button>
                        </td>
                    </tr>
                </template>
            </tbody>
        </table>
    </div>

    <!-- Tab Content: Customers (org's customers) -->
    <div class="tab-content" x-show="activeTab === 'customers'">
        <!-- Similar structure for customers list -->
        <div class="table-header">
            <h3>Kunden dieser Organisation</h3>
            <span class="count" x-text="`${customers.length} Kunden`"></span>
        </div>

        <table class="data-table">
            <thead>
                <tr>
                    <th>Kunde</th>
                    <th>Typ</th>
                    <th>Projekte</th>
                    <th>Umsatz (gesamt)</th>
                    <th>Erstellt</th>
                    <th>Aktionen</th>
                </tr>
            </thead>
            <tbody>
                <template x-for="customer in customers" :key="customer.id">
                    <tr>
                        <td>
                            <div class="customer-cell">
                                <strong x-text="customer.name"></strong>
                                <span class="sub" x-text="customer.email"></span>
                            </div>
                        </td>
                        <td x-text="customer.type"></td>
                        <td x-text="customer.projects_count"></td>
                        <td x-text="formatCurrency(customer.total_revenue)"></td>
                        <td x-text="formatDate(customer.created_at)"></td>
                        <td>
                            <a :href="`/sysadmin/customers/${customer.id}`" class="btn btn-sm">
                                Details
                            </a>
                        </td>
                    </tr>
                </template>
            </tbody>
        </table>
    </div>

    <!-- Tab Content: Projects -->
    <div class="tab-content" x-show="activeTab === 'projects'">
        <!-- Projects list for this organization -->
    </div>

    <!-- Tab Content: Billing -->
    <div class="tab-content" x-show="activeTab === 'billing'">
        <div class="billing-overview">
            <div class="billing-card">
                <h3>Aktueller Plan</h3>
                <div class="plan-info">
                    <span class="plan-name" x-text="organization.plan"></span>
                    <span class="plan-price" x-text="formatCurrency(organization.plan_price)">
                    </span>/Monat
                </div>
                <button @click="openChangePlanModal" class="btn btn-secondary">
                    Plan ändern
                </button>
            </div>

            <div class="billing-card">
                <h3>Module</h3>
                <ul class="module-list">
                    <template x-for="module in organization.modules" :key="module.id">
                        <li>
                            <span x-text="module.name"></span>
                            <span x-text="formatCurrency(module.price)"></span>
                        </li>
                    </template>
                </ul>
                <button @click="openManageModulesModal" class="btn btn-secondary">
                    Module verwalten
                </button>
            </div>

            <div class="billing-card">
                <h3>Gutschriften & Rabatte</h3>
                <div class="credits-info">
                    <span>Aktuelles Guthaben:</span>
                    <span class="credits-amount" x-text="formatCurrency(organization.credits)"></span>
                </div>
                <button @click="openAddCreditModal" class="btn btn-secondary">
                    Guthaben hinzufügen
                </button>
            </div>
        </div>

        <!-- Invoice History -->
        <div class="invoice-history">
            <h3>Rechnungshistorie</h3>
            <table class="data-table">
                <thead>
                    <tr>
                        <th>Rechnungsnr.</th>
                        <th>Datum</th>
                        <th>Betrag</th>
                        <th>Status</th>
                        <th>Aktionen</th>
                    </tr>
                </thead>
                <tbody>
                    <template x-for="invoice in organization.invoices" :key="invoice.id">
                        <tr>
                            <td x-text="invoice.number"></td>
                            <td x-text="formatDate(invoice.date)"></td>
                            <td x-text="formatCurrency(invoice.amount)"></td>
                            <td>
                                <span class="badge" :class="getInvoiceStatusBadge(invoice.status)"
                                      x-text="invoice.status"></span>
                            </td>
                            <td>
                                <a :href="invoice.pdf_url" target="_blank" class="btn btn-sm">
                                    PDF
                                </a>
                            </td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>
    </div>

    <!-- Tab Content: Modules -->
    <div class="tab-content" x-show="activeTab === 'modules'">
        <div class="modules-grid">
            <template x-for="module in allModules" :key="module.id">
                <div class="module-card" :class="{ 'enabled': isModuleEnabled(module.id) }">
                    <div class="module-header">
                        <svg class="module-icon"><use :href="`#icon-${module.icon}`"/></svg>
                        <h4 x-text="module.name"></h4>
                    </div>
                    <p class="module-description" x-text="module.description"></p>
                    <div class="module-footer">
                        <span class="module-price" x-text="formatCurrency(module.price)"></span>
                        <button @click="toggleModule(module.id)"
                                :class="isModuleEnabled(module.id) ? 'btn btn-danger' : 'btn btn-primary'"
                                x-text="isModuleEnabled(module.id) ? 'Deaktivieren' : 'Aktivieren'">
                        </button>
                    </div>
                </div>
            </template>
        </div>
    </div>

    <!-- Tab Content: Settings (org-specific overrides) -->
    <div class="tab-content" x-show="activeTab === 'settings'">
        <form @submit.prevent="saveOrgSettings" class="settings-form">
            <div class="settings-section">
                <h3>Limits</h3>

                <div class="form-group">
                    <label>Max. Benutzer</label>
                    <input type="number" x-model="settings.limits.max_users" min="1">
                    <span class="help">0 = unbegrenzt</span>
                </div>

                <div class="form-group">
                    <label>Max. Projekte</label>
                    <input type="number" x-model="settings.limits.max_projects" min="1">
                </div>

                <div class="form-group">
                    <label>Max. Speicher (GB)</label>
                    <input type="number" x-model="settings.limits.max_storage_gb" min="1">
                </div>

                <div class="form-group">
                    <label>API Rate Limit (req/min)</label>
                    <input type="number" x-model="settings.limits.api_rate_limit" min="10">
                </div>
            </div>

            <div class="settings-section">
                <h3>Feature-Flags</h3>

                <template x-for="flag in featureFlags" :key="flag.key">
                    <div class="form-group-checkbox">
                        <input type="checkbox" :id="flag.key"
                               x-model="settings.features[flag.key]">
                        <label :for="flag.key">
                            <span x-text="flag.label"></span>
                            <span class="help" x-text="flag.description"></span>
                        </label>
                    </div>
                </template>
            </div>

            <div class="settings-section">
                <h3>Branding</h3>

                <div class="form-group">
                    <label>Eigenes Logo erlauben</label>
                    <input type="checkbox" x-model="settings.branding.custom_logo">
                </div>

                <div class="form-group">
                    <label>White-Label (Logo entfernen)</label>
                    <input type="checkbox" x-model="settings.branding.white_label">
                </div>

                <div class="form-group">
                    <label>Eigene Domain</label>
                    <input type="text" x-model="settings.branding.custom_domain"
                           placeholder="kunden.example.de">
                </div>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- Tab Content: Logs -->
    <div class="tab-content" x-show="activeTab === 'logs'">
        <div class="logs-filters">
            <select x-model="logsFilter.type" @change="loadLogs">
                <option value="">Alle Typen</option>
                <option value="auth">Authentifizierung</option>
                <option value="data">Datenänderungen</option>
                <option value="billing">Abrechnung</option>
                <option value="api">API</option>
            </select>
            <input type="date" x-model="logsFilter.from" @change="loadLogs">
            <input type="date" x-model="logsFilter.to" @change="loadLogs">
        </div>

        <div class="logs-list">
            <template x-for="log in logs" :key="log.id">
                <div class="log-entry" :class="`log-${log.level}`">
                    <time x-text="formatDateTime(log.timestamp)"></time>
                    <span class="log-type" x-text="log.type"></span>
                    <span class="log-user" x-text="log.user_name || 'System'"></span>
                    <span class="log-message" x-text="log.message"></span>
                    <button @click="showLogDetails(log)" class="btn-icon-sm">
                        <svg><use href="#icon-eye"/></svg>
                    </button>
                </div>
            </template>
        </div>
    </div>
</div>
```

## Cross-Organisation Kunden- und Auftragsverwaltung

### Alle Kunden (systemweit)

```html
<!-- /templates/sysadmin/customers/list.html -->
<div class="page-all-customers" x-data="allCustomersPage()">
    <header class="page-header">
        <h1>Alle Kunden (systemweit)</h1>
        <p class="subtitle">Kunden aller Organisationen</p>
    </header>

    <div class="filters-bar">
        <input type="search" placeholder="Kunde suchen..."
               x-model="filters.search" @input.debounce.300ms="applyFilters">

        <!-- Organization Filter -->
        <select x-model="filters.organization_id" @change="applyFilters">
            <option value="">Alle Organisationen</option>
            <template x-for="org in organizations" :key="org.id">
                <option :value="org.id" x-text="org.name"></option>
            </template>
        </select>

        <select x-model="filters.customer_type" @change="applyFilters">
            <option value="">Alle Typen</option>
            <option value="private">Privat</option>
            <option value="business">Geschäftlich</option>
            <option value="public">Öffentlich</option>
        </select>

        <button @click="exportCustomers" class="btn btn-secondary">
            Export CSV
        </button>
    </div>

    <table class="data-table">
        <thead>
            <tr>
                <th>Kunde</th>
                <th>Organisation</th>
                <th>Typ</th>
                <th>Projekte</th>
                <th>Offene Aufträge</th>
                <th>Gesamtumsatz</th>
                <th>Erstellt</th>
                <th>Aktionen</th>
            </tr>
        </thead>
        <tbody>
            <template x-for="customer in customers" :key="customer.id">
                <tr>
                    <td>
                        <div class="customer-cell">
                            <strong x-text="customer.name"></strong>
                            <span x-text="customer.email"></span>
                        </div>
                    </td>
                    <td>
                        <a :href="`/sysadmin/organizations/${customer.organization_id}`"
                           x-text="customer.organization_name" class="org-link"></a>
                    </td>
                    <td x-text="customer.type"></td>
                    <td x-text="customer.projects_count"></td>
                    <td x-text="customer.open_orders_count"></td>
                    <td x-text="formatCurrency(customer.total_revenue)"></td>
                    <td x-text="formatDate(customer.created_at)"></td>
                    <td>
                        <a :href="`/sysadmin/customers/${customer.id}`" class="btn btn-sm">
                            Details
                        </a>
                    </td>
                </tr>
            </template>
        </tbody>
    </table>

    <div class="pagination">
        <!-- Pagination controls -->
    </div>
</div>
```

### Alle Aufträge (systemweit)

```html
<!-- /templates/sysadmin/orders/list.html -->
<div class="page-all-orders" x-data="allOrdersPage()">
    <header class="page-header">
        <h1>Alle Aufträge (systemweit)</h1>
    </header>

    <div class="filters-bar">
        <input type="search" placeholder="Auftrag suchen..."
               x-model="filters.search" @input.debounce.300ms="applyFilters">

        <select x-model="filters.organization_id" @change="applyFilters">
            <option value="">Alle Organisationen</option>
            <template x-for="org in organizations" :key="org.id">
                <option :value="org.id" x-text="org.name"></option>
            </template>
        </select>

        <select x-model="filters.status" @change="applyFilters">
            <option value="">Alle Status</option>
            <option value="draft">Entwurf</option>
            <option value="confirmed">Bestätigt</option>
            <option value="in_progress">In Bearbeitung</option>
            <option value="completed">Abgeschlossen</option>
            <option value="cancelled">Storniert</option>
        </select>

        <select x-model="filters.order_type" @change="applyFilters">
            <option value="">Alle Arten</option>
            <option value="energy_consulting">Energieberatung</option>
            <option value="energy_audit">Energieaudit</option>
            <option value="certificate">Energieausweis</option>
            <option value="funding">Fördermittelberatung</option>
        </select>

        <input type="date" x-model="filters.date_from" @change="applyFilters">
        <input type="date" x-model="filters.date_to" @change="applyFilters">
    </div>

    <table class="data-table">
        <thead>
            <tr>
                <th>Auftragsnr.</th>
                <th>Organisation</th>
                <th>Kunde</th>
                <th>Art</th>
                <th>Status</th>
                <th>Wert</th>
                <th>Erstellt</th>
                <th>Fällig</th>
                <th>Aktionen</th>
            </tr>
        </thead>
        <tbody>
            <template x-for="order in orders" :key="order.id">
                <tr :class="{ 'row-overdue': isOverdue(order) }">
                    <td>
                        <a :href="`/sysadmin/orders/${order.id}`" x-text="order.order_number"></a>
                    </td>
                    <td>
                        <a :href="`/sysadmin/organizations/${order.organization_id}`"
                           x-text="order.organization_name"></a>
                    </td>
                    <td x-text="order.customer_name"></td>
                    <td x-text="order.order_type"></td>
                    <td>
                        <span class="badge" :class="getOrderStatusBadge(order.status)"
                              x-text="getOrderStatusLabel(order.status)"></span>
                    </td>
                    <td x-text="formatCurrency(order.total_value)"></td>
                    <td x-text="formatDate(order.created_at)"></td>
                    <td x-text="formatDate(order.due_date)"></td>
                    <td>
                        <a :href="`/sysadmin/orders/${order.id}`" class="btn btn-sm">Details</a>
                    </td>
                </tr>
            </template>
        </tbody>
    </table>
</div>
```

## Systemkonfiguration

### Globale Einstellungen

```html
<!-- /templates/sysadmin/system/config.html -->
<div class="page-system-config" x-data="systemConfigPage()">
    <header class="page-header">
        <h1>Systemkonfiguration</h1>
        <p class="subtitle">Globale Einstellungen für die gesamte Plattform</p>
    </header>

    <div class="config-tabs">
        <button @click="activeSection = 'general'"
                :class="{ 'active': activeSection === 'general' }">Allgemein</button>
        <button @click="activeSection = 'security'"
                :class="{ 'active': activeSection === 'security' }">Sicherheit</button>
        <button @click="activeSection = 'email'"
                :class="{ 'active': activeSection === 'email' }">E-Mail</button>
        <button @click="activeSection = 'storage'"
                :class="{ 'active': activeSection === 'storage' }">Speicher</button>
        <button @click="activeSection = 'ai'"
                :class="{ 'active': activeSection === 'ai' }">KI</button>
        <button @click="activeSection = 'appearance'"
                :class="{ 'active': activeSection === 'appearance' }">Aussehen</button>
        <button @click="activeSection = 'defaults'"
                :class="{ 'active': activeSection === 'defaults' }">Standards</button>
    </div>

    <!-- General Settings -->
    <div class="config-section" x-show="activeSection === 'general'">
        <form @submit.prevent="saveConfig('general')">
            <div class="form-group">
                <label>Plattform-Name</label>
                <input type="text" x-model="config.general.platform_name"
                       placeholder="EnergieberaterPRO">
            </div>

            <div class="form-group">
                <label>Plattform-URL</label>
                <input type="url" x-model="config.general.platform_url"
                       placeholder="https://app.energieberaterpro.de">
            </div>

            <div class="form-group">
                <label>Support-E-Mail</label>
                <input type="email" x-model="config.general.support_email"
                       placeholder="support@energieberaterpro.de">
            </div>

            <div class="form-group">
                <label>Standard-Sprache</label>
                <select x-model="config.general.default_language">
                    <option value="de">Deutsch</option>
                    <option value="en">English</option>
                </select>
            </div>

            <div class="form-group">
                <label>Zeitzone</label>
                <select x-model="config.general.timezone">
                    <option value="Europe/Berlin">Europe/Berlin</option>
                    <option value="Europe/Vienna">Europe/Vienna</option>
                    <option value="Europe/Zurich">Europe/Zurich</option>
                </select>
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="maintenance_mode"
                       x-model="config.general.maintenance_mode">
                <label for="maintenance_mode">
                    <span>Wartungsmodus</span>
                    <span class="help">Nur Sysadmins können auf die Plattform zugreifen</span>
                </label>
            </div>

            <div class="form-group" x-show="config.general.maintenance_mode">
                <label>Wartungsnachricht</label>
                <textarea x-model="config.general.maintenance_message" rows="3"
                          placeholder="Die Plattform wird gerade gewartet..."></textarea>
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="registration_enabled"
                       x-model="config.general.registration_enabled">
                <label for="registration_enabled">
                    <span>Registrierung erlauben</span>
                    <span class="help">Neue Organisationen können sich anmelden</span>
                </label>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- Security Settings -->
    <div class="config-section" x-show="activeSection === 'security'">
        <form @submit.prevent="saveConfig('security')">
            <h3>Passwort-Richtlinien</h3>

            <div class="form-group">
                <label>Mindestlänge</label>
                <input type="number" x-model="config.security.password_min_length" min="8" max="128">
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="require_uppercase"
                       x-model="config.security.password_require_uppercase">
                <label for="require_uppercase">Großbuchstaben erforderlich</label>
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="require_numbers"
                       x-model="config.security.password_require_numbers">
                <label for="require_numbers">Zahlen erforderlich</label>
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="require_special"
                       x-model="config.security.password_require_special">
                <label for="require_special">Sonderzeichen erforderlich</label>
            </div>

            <h3>Session-Einstellungen</h3>

            <div class="form-group">
                <label>Session-Timeout (Minuten)</label>
                <input type="number" x-model="config.security.session_timeout_minutes" min="5">
            </div>

            <div class="form-group">
                <label>Max. gleichzeitige Sessions</label>
                <input type="number" x-model="config.security.max_concurrent_sessions" min="1">
            </div>

            <h3>2-Faktor-Authentifizierung</h3>

            <div class="form-group">
                <label>2FA-Anforderung</label>
                <select x-model="config.security.mfa_requirement">
                    <option value="optional">Optional</option>
                    <option value="required_admins">Pflicht für Admins</option>
                    <option value="required_all">Pflicht für alle</option>
                </select>
            </div>

            <h3>Rate Limiting</h3>

            <div class="form-group">
                <label>Login-Versuche (max)</label>
                <input type="number" x-model="config.security.max_login_attempts" min="3">
            </div>

            <div class="form-group">
                <label>Sperrzeit nach Fehlversuchen (Minuten)</label>
                <input type="number" x-model="config.security.lockout_duration_minutes" min="5">
            </div>

            <div class="form-group">
                <label>API Rate Limit (Standard)</label>
                <input type="number" x-model="config.security.default_api_rate_limit" min="10">
                <span class="help">Anfragen pro Minute</span>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- Email Settings -->
    <div class="config-section" x-show="activeSection === 'email'">
        <form @submit.prevent="saveConfig('email')">
            <h3>SMTP-Konfiguration</h3>

            <div class="form-group">
                <label>SMTP-Server</label>
                <input type="text" x-model="config.email.smtp_host"
                       placeholder="smtp.example.com">
            </div>

            <div class="form-row">
                <div class="form-group">
                    <label>Port</label>
                    <input type="number" x-model="config.email.smtp_port" placeholder="587">
                </div>

                <div class="form-group">
                    <label>Verschlüsselung</label>
                    <select x-model="config.email.smtp_encryption">
                        <option value="tls">TLS</option>
                        <option value="ssl">SSL</option>
                        <option value="none">Keine</option>
                    </select>
                </div>
            </div>

            <div class="form-group">
                <label>Benutzername</label>
                <input type="text" x-model="config.email.smtp_username">
            </div>

            <div class="form-group">
                <label>Passwort</label>
                <input type="password" x-model="config.email.smtp_password"
                       placeholder="••••••••">
            </div>

            <div class="form-group">
                <label>Absender-Name</label>
                <input type="text" x-model="config.email.from_name"
                       placeholder="EnergieberaterPRO">
            </div>

            <div class="form-group">
                <label>Absender-E-Mail</label>
                <input type="email" x-model="config.email.from_email"
                       placeholder="noreply@energieberaterpro.de">
            </div>

            <div class="form-actions">
                <button type="button" @click="testEmail" class="btn btn-secondary">
                    Test-E-Mail senden
                </button>
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- Storage Settings -->
    <div class="config-section" x-show="activeSection === 'storage'">
        <form @submit.prevent="saveConfig('storage')">
            <h3>Speicher-Backend</h3>

            <div class="form-group">
                <label>Speicher-Typ</label>
                <select x-model="config.storage.type">
                    <option value="local">Lokal (Dateisystem)</option>
                    <option value="s3">S3-kompatibel (MinIO, AWS)</option>
                </select>
            </div>

            <div x-show="config.storage.type === 'local'">
                <div class="form-group">
                    <label>Speicherpfad</label>
                    <input type="text" x-model="config.storage.local_path"
                           placeholder="/data/uploads">
                </div>
            </div>

            <div x-show="config.storage.type === 's3'">
                <div class="form-group">
                    <label>S3 Endpoint</label>
                    <input type="url" x-model="config.storage.s3_endpoint"
                           placeholder="https://s3.eu-central-1.amazonaws.com">
                </div>

                <div class="form-group">
                    <label>Bucket</label>
                    <input type="text" x-model="config.storage.s3_bucket"
                           placeholder="energieberaterpro-files">
                </div>

                <div class="form-group">
                    <label>Access Key</label>
                    <input type="text" x-model="config.storage.s3_access_key">
                </div>

                <div class="form-group">
                    <label>Secret Key</label>
                    <input type="password" x-model="config.storage.s3_secret_key">
                </div>
            </div>

            <h3>Limits</h3>

            <div class="form-group">
                <label>Max. Dateigröße (MB)</label>
                <input type="number" x-model="config.storage.max_file_size_mb" min="1">
            </div>

            <div class="form-group">
                <label>Erlaubte Dateitypen</label>
                <input type="text" x-model="config.storage.allowed_extensions"
                       placeholder=".pdf,.jpg,.png,.doc,.docx,.xls,.xlsx">
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- AI Settings -->
    <div class="config-section" x-show="activeSection === 'ai'">
        <form @submit.prevent="saveConfig('ai')">
            <h3>LLM-Konfiguration (Ollama)</h3>

            <div class="form-group">
                <label>Ollama-URL</label>
                <input type="url" x-model="config.ai.ollama_url"
                       placeholder="http://ollama:11434">
            </div>

            <div class="form-group">
                <label>Standard-Modell</label>
                <select x-model="config.ai.default_model">
                    <option value="llama3.2">Llama 3.2 (8B)</option>
                    <option value="llama3.2:70b">Llama 3.2 (70B)</option>
                    <option value="mistral">Mistral (7B)</option>
                    <option value="mixtral">Mixtral (8x7B)</option>
                    <option value="codestral">Codestral</option>
                </select>
            </div>

            <div class="form-group">
                <label>Max. Tokens pro Anfrage</label>
                <input type="number" x-model="config.ai.max_tokens" min="256" max="32768">
            </div>

            <h3>Vector-Datenbank (ChromaDB)</h3>

            <div class="form-group">
                <label>ChromaDB-URL</label>
                <input type="url" x-model="config.ai.chromadb_url"
                       placeholder="http://chromadb:8000">
            </div>

            <div class="form-group">
                <label>Embedding-Modell</label>
                <select x-model="config.ai.embedding_model">
                    <option value="all-MiniLM-L6-v2">all-MiniLM-L6-v2</option>
                    <option value="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2">
                        Multilingual MiniLM
                    </option>
                </select>
            </div>

            <h3>Feature-Kontrolle</h3>

            <div class="form-group-checkbox">
                <input type="checkbox" id="ai_enabled" x-model="config.ai.enabled">
                <label for="ai_enabled">KI-Features aktivieren</label>
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="ai_chat_enabled" x-model="config.ai.chat_enabled">
                <label for="ai_chat_enabled">KI-Chat aktivieren</label>
            </div>

            <div class="form-group-checkbox">
                <input type="checkbox" id="ai_reports_enabled" x-model="config.ai.reports_enabled">
                <label for="ai_reports_enabled">KI-Berichterstellung aktivieren</label>
            </div>

            <div class="form-actions">
                <button type="button" @click="testAI" class="btn btn-secondary">
                    Verbindung testen
                </button>
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- Appearance Settings -->
    <div class="config-section" x-show="activeSection === 'appearance'">
        <form @submit.prevent="saveConfig('appearance')">
            <h3>Branding</h3>

            <div class="form-group">
                <label>Logo (hell)</label>
                <div class="file-upload">
                    <img :src="config.appearance.logo_light" alt="" class="preview">
                    <input type="file" @change="uploadLogo('light', $event)" accept="image/*">
                </div>
            </div>

            <div class="form-group">
                <label>Logo (dunkel)</label>
                <div class="file-upload">
                    <img :src="config.appearance.logo_dark" alt="" class="preview">
                    <input type="file" @change="uploadLogo('dark', $event)" accept="image/*">
                </div>
            </div>

            <div class="form-group">
                <label>Favicon</label>
                <div class="file-upload">
                    <img :src="config.appearance.favicon" alt="" class="preview preview-sm">
                    <input type="file" @change="uploadFavicon($event)" accept="image/*">
                </div>
            </div>

            <h3>Farben</h3>

            <div class="color-grid">
                <div class="form-group">
                    <label>Primärfarbe</label>
                    <input type="color" x-model="config.appearance.primary_color">
                </div>

                <div class="form-group">
                    <label>Sekundärfarbe</label>
                    <input type="color" x-model="config.appearance.secondary_color">
                </div>

                <div class="form-group">
                    <label>Akzentfarbe</label>
                    <input type="color" x-model="config.appearance.accent_color">
                </div>
            </div>

            <h3>Login-Seite</h3>

            <div class="form-group">
                <label>Hintergrundbild</label>
                <div class="file-upload">
                    <img :src="config.appearance.login_background" alt="" class="preview preview-lg">
                    <input type="file" @change="uploadLoginBg($event)" accept="image/*">
                </div>
            </div>

            <div class="form-group">
                <label>Willkommensnachricht</label>
                <textarea x-model="config.appearance.login_message" rows="3"></textarea>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>

    <!-- Defaults Settings -->
    <div class="config-section" x-show="activeSection === 'defaults'">
        <form @submit.prevent="saveConfig('defaults')">
            <h3>Standard-Plan für neue Organisationen</h3>

            <div class="form-group">
                <label>Plan</label>
                <select x-model="config.defaults.default_plan">
                    <option value="trial">Testversion (14 Tage)</option>
                    <option value="starter">Starter</option>
                    <option value="professional">Professional</option>
                </select>
            </div>

            <div class="form-group">
                <label>Trial-Dauer (Tage)</label>
                <input type="number" x-model="config.defaults.trial_days" min="1" max="90">
            </div>

            <h3>Standard-Limits</h3>

            <div class="form-group">
                <label>Max. Benutzer (Standard)</label>
                <input type="number" x-model="config.defaults.max_users" min="1">
            </div>

            <div class="form-group">
                <label>Max. Projekte (Standard)</label>
                <input type="number" x-model="config.defaults.max_projects" min="1">
            </div>

            <div class="form-group">
                <label>Max. Speicher GB (Standard)</label>
                <input type="number" x-model="config.defaults.max_storage_gb" min="1">
            </div>

            <h3>Standard-Module</h3>

            <div class="modules-checkbox-list">
                <template x-for="module in allModules" :key="module.id">
                    <div class="form-group-checkbox">
                        <input type="checkbox" :id="`default_module_${module.id}`"
                               :value="module.id" x-model="config.defaults.default_modules">
                        <label :for="`default_module_${module.id}`" x-text="module.name"></label>
                    </div>
                </template>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Speichern</button>
            </div>
        </form>
    </div>
</div>
```

## Module & Feature-Verwaltung

```html
<!-- /templates/sysadmin/modules/list.html -->
<div class="page-modules" x-data="modulesPage()">
    <header class="page-header">
        <h1>Module & Features</h1>
        <div class="header-actions">
            <button @click="openAddModuleModal" class="btn btn-primary">
                + Neues Modul
            </button>
        </div>
    </header>

    <div class="modules-list">
        <template x-for="module in modules" :key="module.id">
            <div class="module-card-admin">
                <div class="module-header">
                    <div class="module-info">
                        <svg class="module-icon"><use :href="`#icon-${module.icon}`"/></svg>
                        <div>
                            <h3 x-text="module.name"></h3>
                            <span class="module-key" x-text="module.key"></span>
                        </div>
                    </div>
                    <div class="module-status">
                        <span class="badge" :class="module.active ? 'badge-success' : 'badge-secondary'"
                              x-text="module.active ? 'Aktiv' : 'Inaktiv'"></span>
                    </div>
                </div>

                <p class="module-description" x-text="module.description"></p>

                <div class="module-stats">
                    <div class="stat">
                        <span class="stat-value" x-text="module.organizations_count"></span>
                        <span class="stat-label">Organisationen</span>
                    </div>
                    <div class="stat">
                        <span class="stat-value" x-text="formatCurrency(module.monthly_revenue)"></span>
                        <span class="stat-label">Monatl. Umsatz</span>
                    </div>
                </div>

                <div class="module-pricing">
                    <h4>Preisgestaltung</h4>
                    <table class="pricing-table">
                        <thead>
                            <tr>
                                <th>Plan</th>
                                <th>Preis/Monat</th>
                                <th>Enthalten</th>
                            </tr>
                        </thead>
                        <tbody>
                            <template x-for="pricing in module.pricing" :key="pricing.plan">
                                <tr>
                                    <td x-text="pricing.plan"></td>
                                    <td x-text="pricing.included ? 'Inklusive' : formatCurrency(pricing.price)"></td>
                                    <td>
                                        <input type="checkbox" :checked="pricing.included"
                                               @change="toggleIncluded(module.id, pricing.plan)">
                                    </td>
                                </tr>
                            </template>
                        </tbody>
                    </table>
                </div>

                <div class="module-features">
                    <h4>Features</h4>
                    <ul class="feature-list">
                        <template x-for="feature in module.features" :key="feature.key">
                            <li>
                                <span x-text="feature.name"></span>
                                <button @click="editFeature(module.id, feature.key)"
                                        class="btn-icon-sm">
                                    <svg><use href="#icon-edit"/></svg>
                                </button>
                            </li>
                        </template>
                    </ul>
                    <button @click="addFeature(module.id)" class="btn btn-sm">
                        + Feature
                    </button>
                </div>

                <div class="module-actions">
                    <button @click="editModule(module.id)" class="btn btn-secondary">
                        Bearbeiten
                    </button>
                    <button @click="toggleModuleStatus(module.id)"
                            :class="module.active ? 'btn btn-warning' : 'btn btn-success'"
                            x-text="module.active ? 'Deaktivieren' : 'Aktivieren'">
                    </button>
                </div>
            </div>
        </template>
    </div>
</div>
```

## JavaScript Controller

```javascript
// /static/js/sysadmin/dashboard.js

function sysadminDashboard() {
    return {
        metrics: {
            organizations: { total: 0, change: 0 },
            users: { total: 0, active: 0 },
            projects: { total: 0, active: 0 },
            revenue: { monthly: 0, change: 0 },
            tickets: { open: 0 },
            pending: { organizations: 0 },
            jobs: { failed: 0 },
            storage: { used: 0 },
            api: { rpm: 0 }
        },
        recentActivity: [],
        systemAlerts: [],
        charts: {},

        async init() {
            await this.loadMetrics();
            await this.loadActivity();
            await this.loadAlerts();
            this.initCharts();

            // Auto-refresh every 30 seconds
            setInterval(() => this.loadMetrics(), 30000);
        },

        async loadMetrics() {
            const response = await fetch('/api/v1/sysadmin/dashboard/metrics');
            this.metrics = await response.json();
        },

        async loadActivity() {
            const response = await fetch('/api/v1/sysadmin/dashboard/activity?limit=10');
            this.recentActivity = await response.json();
        },

        async loadAlerts() {
            const response = await fetch('/api/v1/sysadmin/dashboard/alerts');
            this.systemAlerts = await response.json();
        },

        initCharts() {
            // User Activity Chart
            const userCtx = document.getElementById('userActivityChart').getContext('2d');
            this.charts.userActivity = new Chart(userCtx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Aktive Benutzer',
                        data: [],
                        borderColor: '#3b82f6',
                        tension: 0.4
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: { display: false }
                    }
                }
            });

            this.loadChartData();
        },

        async loadChartData() {
            const response = await fetch('/api/v1/sysadmin/dashboard/charts/user-activity');
            const data = await response.json();

            this.charts.userActivity.data.labels = data.labels;
            this.charts.userActivity.data.datasets[0].data = data.values;
            this.charts.userActivity.update();
        },

        async dismissAlert(alertId) {
            await fetch(`/api/v1/sysadmin/alerts/${alertId}/dismiss`, { method: 'POST' });
            this.systemAlerts = this.systemAlerts.filter(a => a.id !== alertId);
        },

        formatCurrency(amount) {
            return new Intl.NumberFormat('de-DE', {
                style: 'currency',
                currency: 'EUR'
            }).format(amount);
        },

        formatTime(timestamp) {
            return new Date(timestamp).toLocaleTimeString('de-DE', {
                hour: '2-digit',
                minute: '2-digit'
            });
        }
    };
}

function organizationsPage() {
    return {
        organizations: [],
        filters: {
            search: '',
            status: '',
            plan: '',
            sortBy: 'created_desc'
        },
        pagination: {
            page: 1,
            perPage: 25,
            total: 0,
            from: 0,
            to: 0,
            lastPage: 1
        },
        selected: [],
        openMenu: null,

        get selectedAll() {
            return this.selected.length === this.organizations.length && this.organizations.length > 0;
        },

        get selectedSome() {
            return this.selected.length > 0 && this.selected.length < this.organizations.length;
        },

        async init() {
            await this.loadOrganizations();
        },

        async loadOrganizations() {
            const params = new URLSearchParams({
                ...this.filters,
                page: this.pagination.page,
                per_page: this.pagination.perPage
            });

            const response = await fetch(`/api/v1/sysadmin/organizations?${params}`);
            const data = await response.json();

            this.organizations = data.items;
            this.pagination = {
                ...this.pagination,
                total: data.total,
                from: data.from,
                to: data.to,
                lastPage: data.last_page
            };
        },

        applyFilters() {
            this.pagination.page = 1;
            this.loadOrganizations();
        },

        toggleSelectAll(event) {
            if (event.target.checked) {
                this.selected = this.organizations.map(o => o.id);
            } else {
                this.selected = [];
            }
        },

        async impersonateAdmin(orgId) {
            const response = await fetch(`/api/v1/sysadmin/organizations/${orgId}/impersonate`, {
                method: 'POST'
            });
            const data = await response.json();

            if (data.success) {
                // Open in new tab with impersonation token
                window.open(data.redirect_url, '_blank');
            }
        },

        async suspendOrganization(orgId) {
            if (!confirm('Organisation wirklich sperren?')) return;

            const reason = prompt('Grund für die Sperrung:');
            if (!reason) return;

            await fetch(`/api/v1/sysadmin/organizations/${orgId}/suspend`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ reason })
            });

            this.loadOrganizations();
        },

        async activateOrganization(orgId) {
            await fetch(`/api/v1/sysadmin/organizations/${orgId}/activate`, {
                method: 'POST'
            });
            this.loadOrganizations();
        },

        getStatusColor(status) {
            const colors = {
                active: 'success',
                trial: 'info',
                suspended: 'danger',
                pending: 'warning'
            };
            return colors[status] || 'secondary';
        },

        getStatusLabel(status) {
            const labels = {
                active: 'Aktiv',
                trial: 'Testversion',
                suspended: 'Gesperrt',
                pending: 'Ausstehend'
            };
            return labels[status] || status;
        },

        formatCurrency(amount) {
            return new Intl.NumberFormat('de-DE', {
                style: 'currency',
                currency: 'EUR'
            }).format(amount);
        },

        formatDate(dateStr) {
            return new Date(dateStr).toLocaleDateString('de-DE');
        }
    };
}

function systemConfigPage() {
    return {
        activeSection: 'general',
        config: {
            general: {},
            security: {},
            email: {},
            storage: {},
            ai: {},
            appearance: {},
            defaults: {}
        },
        allModules: [],

        async init() {
            await this.loadConfig();
            await this.loadModules();
        },

        async loadConfig() {
            const response = await fetch('/api/v1/sysadmin/config');
            this.config = await response.json();
        },

        async loadModules() {
            const response = await fetch('/api/v1/sysadmin/modules');
            this.allModules = await response.json();
        },

        async saveConfig(section) {
            const response = await fetch(`/api/v1/sysadmin/config/${section}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(this.config[section])
            });

            if (response.ok) {
                this.showNotification('Einstellungen gespeichert', 'success');
            } else {
                this.showNotification('Fehler beim Speichern', 'error');
            }
        },

        async testEmail() {
            const response = await fetch('/api/v1/sysadmin/config/email/test', {
                method: 'POST'
            });

            if (response.ok) {
                this.showNotification('Test-E-Mail gesendet', 'success');
            } else {
                this.showNotification('E-Mail-Test fehlgeschlagen', 'error');
            }
        },

        async testAI() {
            const response = await fetch('/api/v1/sysadmin/config/ai/test', {
                method: 'POST'
            });

            const data = await response.json();

            if (data.success) {
                this.showNotification(`Verbindung OK: ${data.model}`, 'success');
            } else {
                this.showNotification(`Fehler: ${data.error}`, 'error');
            }
        },

        showNotification(message, type) {
            // Use global notification system
            window.dispatchEvent(new CustomEvent('notification', {
                detail: { message, type }
            }));
        }
    };
}
```

## API-Endpunkte

```python
# /app/api/v1/sysadmin/routes.py

from fastapi import APIRouter, Depends, HTTPException, Query
from typing import Optional, List
from app.core.auth import require_sysadmin
from app.services.sysadmin import SysadminService
from app.schemas.sysadmin import *

router = APIRouter(prefix="/sysadmin", tags=["sysadmin"])

@router.get("/dashboard/metrics")
async def get_dashboard_metrics(
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Holt Dashboard-Metriken für Sysadmin"""
    return await service.get_dashboard_metrics()


@router.get("/organizations")
async def list_organizations(
    search: Optional[str] = None,
    status: Optional[str] = None,
    plan: Optional[str] = None,
    sort_by: str = "created_desc",
    page: int = Query(1, ge=1),
    per_page: int = Query(25, ge=10, le=100),
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Listet alle Organisationen mit Filterung und Paginierung"""
    return await service.list_organizations(
        search=search,
        status=status,
        plan=plan,
        sort_by=sort_by,
        page=page,
        per_page=per_page
    )


@router.get("/organizations/{org_id}")
async def get_organization(
    org_id: int,
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Holt Details einer Organisation"""
    return await service.get_organization_details(org_id)


@router.post("/organizations/{org_id}/suspend")
async def suspend_organization(
    org_id: int,
    data: SuspendOrganizationRequest,
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Sperrt eine Organisation"""
    return await service.suspend_organization(
        admin_id=current_user.id,
        org_id=org_id,
        reason=data.reason
    )


@router.post("/organizations/{org_id}/impersonate")
async def impersonate_org_admin(
    org_id: int,
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Erstellt Support-Session als Org-Admin"""
    return await service.impersonate_org_admin(
        admin_id=current_user.id,
        org_id=org_id
    )


@router.get("/customers")
async def list_all_customers(
    organization_id: Optional[int] = None,
    search: Optional[str] = None,
    customer_type: Optional[str] = None,
    page: int = Query(1, ge=1),
    per_page: int = Query(25, ge=10, le=100),
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Listet alle Kunden (systemweit, über alle Organisationen)"""
    return await service.list_all_customers(
        organization_id=organization_id,
        search=search,
        customer_type=customer_type,
        page=page,
        per_page=per_page
    )


@router.get("/orders")
async def list_all_orders(
    organization_id: Optional[int] = None,
    status: Optional[str] = None,
    order_type: Optional[str] = None,
    date_from: Optional[str] = None,
    date_to: Optional[str] = None,
    page: int = Query(1, ge=1),
    per_page: int = Query(25, ge=10, le=100),
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Listet alle Aufträge (systemweit, über alle Organisationen)"""
    return await service.list_all_orders(
        organization_id=organization_id,
        status=status,
        order_type=order_type,
        date_from=date_from,
        date_to=date_to,
        page=page,
        per_page=per_page
    )


@router.get("/config")
async def get_system_config(
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Holt komplette Systemkonfiguration"""
    return await service.get_system_config()


@router.put("/config/{section}")
async def update_config_section(
    section: str,
    config: dict,
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Aktualisiert einen Konfigurationsbereich"""
    return await service.update_config_section(
        admin_id=current_user.id,
        section=section,
        config=config
    )


@router.get("/modules")
async def list_modules(
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Listet alle Module mit Statistiken"""
    return await service.list_modules_with_stats()


@router.put("/modules/{module_id}")
async def update_module(
    module_id: int,
    data: UpdateModuleRequest,
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Aktualisiert Modul-Einstellungen"""
    return await service.update_module(
        admin_id=current_user.id,
        module_id=module_id,
        data=data
    )


@router.get("/logs/audit")
async def get_audit_logs(
    log_type: Optional[str] = None,
    user_id: Optional[int] = None,
    organization_id: Optional[int] = None,
    date_from: Optional[str] = None,
    date_to: Optional[str] = None,
    page: int = Query(1, ge=1),
    per_page: int = Query(50, ge=10, le=200),
    current_user = Depends(require_sysadmin),
    service: SysadminService = Depends()
):
    """Holt Audit-Logs"""
    return await service.get_audit_logs(
        log_type=log_type,
        user_id=user_id,
        organization_id=organization_id,
        date_from=date_from,
        date_to=date_to,
        page=page,
        per_page=per_page
    )
```

## CSS Styles

```css
/* /static/css/sysadmin.css */

/* Sysadmin Layout */
.sysadmin-layout {
    display: flex;
    min-height: 100vh;
}

.sysadmin-sidebar {
    width: 260px;
    background: var(--color-gray-900);
    color: white;
    display: flex;
    flex-direction: column;
    position: fixed;
    height: 100vh;
    overflow-y: auto;
}

.sidebar-header {
    padding: 1.5rem;
    display: flex;
    align-items: center;
    gap: 0.75rem;
    border-bottom: 1px solid var(--color-gray-800);
}

.sidebar-header .logo {
    height: 32px;
}

.sidebar-header .badge-danger {
    background: var(--color-red-600);
    color: white;
    font-size: 0.625rem;
    padding: 0.125rem 0.375rem;
    border-radius: 0.25rem;
    font-weight: 600;
}

.sidebar-nav {
    flex: 1;
    padding: 1rem 0;
}

.nav-item,
.nav-group-toggle {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.75rem 1.5rem;
    color: var(--color-gray-300);
    text-decoration: none;
    width: 100%;
    background: none;
    border: none;
    cursor: pointer;
    font-size: 0.875rem;
    transition: background 0.15s, color 0.15s;
}

.nav-item:hover,
.nav-group-toggle:hover {
    background: var(--color-gray-800);
    color: white;
}

.nav-item.active {
    background: var(--color-blue-600);
    color: white;
}

.nav-item .icon,
.nav-group-toggle .icon {
    width: 20px;
    height: 20px;
}

.nav-group-toggle .chevron {
    margin-left: auto;
    width: 16px;
    height: 16px;
    transition: transform 0.2s;
}

.nav-group-toggle .chevron.rotated {
    transform: rotate(90deg);
}

.nav-group-items {
    background: var(--color-gray-850);
}

.nav-group-items a {
    display: block;
    padding: 0.5rem 1.5rem 0.5rem 3.25rem;
    color: var(--color-gray-400);
    text-decoration: none;
    font-size: 0.8125rem;
}

.nav-group-items a:hover {
    color: white;
    background: var(--color-gray-800);
}

/* Main Content */
.sysadmin-main {
    flex: 1;
    margin-left: 260px;
    display: flex;
    flex-direction: column;
}

.sysadmin-header {
    background: white;
    border-bottom: 1px solid var(--color-gray-200);
    padding: 0.75rem 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: sticky;
    top: 0;
    z-index: 100;
}

.sysadmin-content {
    padding: 1.5rem;
    flex: 1;
    background: var(--color-gray-50);
}

/* Metrics Grid */
.metrics-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 1.5rem;
    margin-bottom: 1.5rem;
}

.metric-card {
    background: white;
    border-radius: 0.5rem;
    padding: 1.25rem;
    display: flex;
    align-items: flex-start;
    gap: 1rem;
    box-shadow: var(--shadow-sm);
}

.metric-icon {
    width: 48px;
    height: 48px;
    border-radius: 0.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
}

.metric-icon svg {
    width: 24px;
    height: 24px;
    color: white;
}

.metric-icon.bg-blue { background: var(--color-blue-500); }
.metric-icon.bg-green { background: var(--color-green-500); }
.metric-icon.bg-purple { background: var(--color-purple-500); }
.metric-icon.bg-yellow { background: var(--color-yellow-500); }

.metric-content {
    display: flex;
    flex-direction: column;
}

.metric-label {
    font-size: 0.875rem;
    color: var(--color-gray-500);
}

.metric-value {
    font-size: 1.75rem;
    font-weight: 600;
    color: var(--color-gray-900);
}

.metric-change {
    font-size: 0.75rem;
}

.metric-change.positive { color: var(--color-green-600); }
.metric-change.negative { color: var(--color-red-600); }

/* Data Tables */
.data-table-container {
    background: white;
    border-radius: 0.5rem;
    overflow: hidden;
    box-shadow: var(--shadow-sm);
}

.data-table {
    width: 100%;
    border-collapse: collapse;
}

.data-table th {
    background: var(--color-gray-50);
    padding: 0.75rem 1rem;
    text-align: left;
    font-size: 0.75rem;
    font-weight: 600;
    text-transform: uppercase;
    color: var(--color-gray-600);
    border-bottom: 1px solid var(--color-gray-200);
}

.data-table td {
    padding: 0.875rem 1rem;
    border-bottom: 1px solid var(--color-gray-100);
}

.data-table tr:hover {
    background: var(--color-gray-50);
}

.data-table .row-suspended {
    background: var(--color-red-50);
}

/* Organization Cell */
.org-cell {
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

.org-logo {
    width: 32px;
    height: 32px;
    border-radius: 0.25rem;
    object-fit: cover;
}

.org-name {
    font-weight: 500;
    color: var(--color-gray-900);
    text-decoration: none;
}

.org-name:hover {
    color: var(--color-blue-600);
}

.org-slug {
    font-size: 0.75rem;
    color: var(--color-gray-500);
    display: block;
}

/* Badges */
.badge {
    display: inline-flex;
    align-items: center;
    padding: 0.25rem 0.5rem;
    font-size: 0.75rem;
    font-weight: 500;
    border-radius: 9999px;
}

.badge-success { background: var(--color-green-100); color: var(--color-green-800); }
.badge-warning { background: var(--color-yellow-100); color: var(--color-yellow-800); }
.badge-danger { background: var(--color-red-100); color: var(--color-red-800); }
.badge-info { background: var(--color-blue-100); color: var(--color-blue-800); }
.badge-secondary { background: var(--color-gray-100); color: var(--color-gray-800); }

/* Filters Bar */
.filters-bar {
    display: flex;
    gap: 1rem;
    margin-bottom: 1.5rem;
    flex-wrap: wrap;
}

.filters-bar input,
.filters-bar select {
    padding: 0.5rem 0.75rem;
    border: 1px solid var(--color-gray-300);
    border-radius: 0.375rem;
    font-size: 0.875rem;
}

.search-input {
    position: relative;
    flex: 1;
    max-width: 300px;
}

.search-input .icon {
    position: absolute;
    left: 0.75rem;
    top: 50%;
    transform: translateY(-50%);
    width: 16px;
    height: 16px;
    color: var(--color-gray-400);
}

.search-input input {
    padding-left: 2.25rem;
    width: 100%;
}

/* Config Sections */
.config-section {
    background: white;
    border-radius: 0.5rem;
    padding: 1.5rem;
    margin-top: 1.5rem;
}

.config-section h3 {
    font-size: 1rem;
    font-weight: 600;
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--color-gray-200);
}

.form-group {
    margin-bottom: 1rem;
}

.form-group label {
    display: block;
    font-size: 0.875rem;
    font-weight: 500;
    margin-bottom: 0.375rem;
    color: var(--color-gray-700);
}

.form-group input,
.form-group select,
.form-group textarea {
    width: 100%;
    padding: 0.5rem 0.75rem;
    border: 1px solid var(--color-gray-300);
    border-radius: 0.375rem;
    font-size: 0.875rem;
}

.form-group .help {
    font-size: 0.75rem;
    color: var(--color-gray-500);
    margin-top: 0.25rem;
}

.form-actions {
    margin-top: 1.5rem;
    padding-top: 1rem;
    border-top: 1px solid var(--color-gray-200);
    display: flex;
    gap: 0.75rem;
}

/* Module Cards */
.modules-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 1.5rem;
}

.module-card-admin {
    background: white;
    border-radius: 0.5rem;
    padding: 1.25rem;
    box-shadow: var(--shadow-sm);
}

.module-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 0.75rem;
}

.module-info {
    display: flex;
    gap: 0.75rem;
    align-items: center;
}

.module-icon {
    width: 40px;
    height: 40px;
    background: var(--color-blue-100);
    border-radius: 0.375rem;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--color-blue-600);
}

.module-description {
    font-size: 0.875rem;
    color: var(--color-gray-600);
    margin-bottom: 1rem;
}

.module-stats {
    display: flex;
    gap: 1.5rem;
    margin-bottom: 1rem;
    padding: 0.75rem;
    background: var(--color-gray-50);
    border-radius: 0.375rem;
}

.module-stats .stat-value {
    font-size: 1.25rem;
    font-weight: 600;
    display: block;
}

.module-stats .stat-label {
    font-size: 0.75rem;
    color: var(--color-gray-500);
}

.module-actions {
    display: flex;
    gap: 0.5rem;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid var(--color-gray-200);
}
```

## Zusammenfassung

Die Sysadmin-Oberfläche bietet:

1. **Vollständige Kontrolle**: Zugriff auf ALLE Organisationen, Benutzer, Kunden, Projekte und Aufträge
2. **Cross-Organisation-Verwaltung**: Kunden und Aufträge über alle Organisationen hinweg einsehbar
3. **Systemkonfiguration**: Komplette Plattform-Einstellungen (E-Mail, Speicher, KI, Sicherheit)
4. **Modul-Management**: Feature-Freischaltung pro Organisation
5. **Support-Features**: Impersonation, Audit-Logs, System-Warnungen
6. **Billing-Kontrolle**: Pläne, Preise, Gutscheine, Rechnungen
