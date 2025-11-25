# Org-Admin-Oberfläche

## Übersicht

Die Org-Admin-Oberfläche ermöglicht Organisationsadministratoren die vollständige Verwaltung ihrer eigenen Organisation, ohne Zugriff auf andere Organisationen oder Systemeinstellungen.

## Berechtigungsebene

```
┌────────────────────────────────────────────────────────────────────────────┐
│                           ORG-ADMIN ZUGRIFF                                 │
├────────────────────────────────────────────────────────────────────────────┤
│  ✓ Eigene Organisation verwalten                                           │
│  ✓ Mitarbeiter hinzufügen/entfernen                                        │
│  ✓ Kunden und Projekte der Organisation                                    │
│  ✓ Aufträge und Rechnungen                                                 │
│  ✓ Organisations-Einstellungen (Logo, E-Mail, Templates)                   │
│  ✓ Modul-Nutzung und Abrechnung einsehen                                   │
│  ✗ Andere Organisationen                                                   │
│  ✗ Systemkonfiguration                                                     │
│  ✗ Plattform-weite Einstellungen                                           │
└────────────────────────────────────────────────────────────────────────────┘
```

## Haupt-Navigation

```html
<!-- /templates/orgadmin/layout.html -->
<div class="orgadmin-layout" x-data="orgadminLayout()">
    <!-- Sidebar -->
    <aside class="sidebar">
        <div class="sidebar-header">
            <img :src="organization.logo || '/static/img/logo.svg'" alt="" class="logo">
            <span class="org-name" x-text="organization.name"></span>
        </div>

        <nav class="sidebar-nav">
            <!-- Dashboard -->
            <a href="/admin/dashboard" class="nav-item"
               :class="{ 'active': currentPage === 'dashboard' }">
                <svg class="icon"><use href="#icon-dashboard"/></svg>
                <span>Dashboard</span>
            </a>

            <!-- Mitarbeiter -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('team')">
                    <svg class="icon"><use href="#icon-users"/></svg>
                    <span>Team</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.team }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.team">
                    <a href="/admin/team">Mitarbeiter</a>
                    <a href="/admin/team/invite">Einladen</a>
                    <a href="/admin/team/roles">Rollen</a>
                    <a href="/admin/team/activity">Aktivität</a>
                </div>
            </div>

            <!-- Kunden -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('customers')">
                    <svg class="icon"><use href="#icon-building"/></svg>
                    <span>Kunden</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.customers }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.customers">
                    <a href="/admin/customers">Alle Kunden</a>
                    <a href="/admin/customers/new">Neuer Kunde</a>
                    <a href="/admin/customers/import">Import</a>
                </div>
            </div>

            <!-- Projekte -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('projects')">
                    <svg class="icon"><use href="#icon-folder"/></svg>
                    <span>Projekte</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.projects }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.projects">
                    <a href="/admin/projects">Alle Projekte</a>
                    <a href="/admin/projects/active">Aktive</a>
                    <a href="/admin/projects/templates">Vorlagen</a>
                </div>
            </div>

            <!-- Aufträge -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('orders')">
                    <svg class="icon"><use href="#icon-clipboard"/></svg>
                    <span>Aufträge</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.orders }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.orders">
                    <a href="/admin/orders">Alle Aufträge</a>
                    <a href="/admin/orders/pipeline">Pipeline</a>
                    <a href="/admin/quotes">Angebote</a>
                    <a href="/admin/invoices">Rechnungen</a>
                </div>
            </div>

            <!-- Berichte -->
            <a href="/admin/reports" class="nav-item"
               :class="{ 'active': currentPage === 'reports' }">
                <svg class="icon"><use href="#icon-chart"/></svg>
                <span>Berichte</span>
            </a>

            <!-- Zeiterfassung -->
            <a href="/admin/timetracking" class="nav-item"
               :class="{ 'active': currentPage === 'timetracking' }">
                <svg class="icon"><use href="#icon-clock"/></svg>
                <span>Zeiterfassung</span>
            </a>

            <!-- Einstellungen -->
            <div class="nav-group">
                <button class="nav-group-toggle" @click="toggleGroup('settings')">
                    <svg class="icon"><use href="#icon-cog"/></svg>
                    <span>Einstellungen</span>
                    <svg class="chevron" :class="{ 'rotated': openGroups.settings }">
                        <use href="#icon-chevron"/>
                    </svg>
                </button>
                <div class="nav-group-items" x-show="openGroups.settings">
                    <a href="/admin/settings/organization">Organisation</a>
                    <a href="/admin/settings/branding">Branding</a>
                    <a href="/admin/settings/email">E-Mail</a>
                    <a href="/admin/settings/templates">Vorlagen</a>
                    <a href="/admin/settings/integrations">Integrationen</a>
                    <a href="/admin/settings/billing">Abrechnung</a>
                </div>
            </div>
        </nav>

        <!-- Plan Info -->
        <div class="sidebar-footer">
            <div class="plan-info">
                <span class="plan-name" x-text="organization.plan"></span>
                <div class="usage-bar">
                    <div class="usage-fill" :style="`width: ${usagePercent}%`"></div>
                </div>
                <span class="usage-text" x-text="`${usage.users}/${limits.users} Benutzer`"></span>
            </div>
            <a href="/admin/settings/billing" class="btn btn-sm btn-outline">
                Plan verwalten
            </a>
        </div>
    </aside>

    <!-- Main Content -->
    <main class="main-content">
        <header class="main-header">
            <div class="header-left">
                <button @click="toggleSidebar" class="btn-icon mobile-only">
                    <svg class="icon"><use href="#icon-menu"/></svg>
                </button>
                <div class="breadcrumbs" x-html="breadcrumbs"></div>
            </div>
            <div class="header-right">
                <div class="search-global">
                    <input type="search" placeholder="Suchen..." @keyup.enter="globalSearch">
                    <kbd>⌘K</kbd>
                </div>
                <button class="btn-icon" @click="openNotifications">
                    <svg class="icon"><use href="#icon-bell"/></svg>
                    <span class="badge" x-show="notifications > 0" x-text="notifications"></span>
                </button>
                <div class="user-menu" @click.away="userMenuOpen = false">
                    <button @click="userMenuOpen = !userMenuOpen" class="user-button">
                        <img :src="user.avatar" alt="" class="avatar">
                        <span x-text="user.name"></span>
                        <span class="badge badge-admin">Admin</span>
                    </button>
                    <div class="dropdown" x-show="userMenuOpen">
                        <a href="/profile">Mein Profil</a>
                        <a href="/admin/settings">Einstellungen</a>
                        <hr>
                        <a href="/logout" class="text-danger">Abmelden</a>
                    </div>
                </div>
            </div>
        </header>

        <div class="page-content">
            <!-- Page content injected here -->
        </div>
    </main>
</div>
```

## Dashboard

```html
<!-- /templates/orgadmin/dashboard.html -->
<div class="dashboard" x-data="orgadminDashboard()">
    <header class="page-header">
        <h1>Dashboard</h1>
        <div class="header-actions">
            <select x-model="dateRange" @change="loadData">
                <option value="7d">Letzte 7 Tage</option>
                <option value="30d">Letzte 30 Tage</option>
                <option value="90d">Letzte 90 Tage</option>
                <option value="year">Dieses Jahr</option>
            </select>
        </div>
    </header>

    <!-- Quick Stats -->
    <section class="stats-grid">
        <div class="stat-card">
            <div class="stat-icon bg-blue">
                <svg><use href="#icon-users"/></svg>
            </div>
            <div class="stat-content">
                <span class="stat-label">Aktive Kunden</span>
                <span class="stat-value" x-text="stats.customers.active">0</span>
                <span class="stat-change positive" x-show="stats.customers.change > 0">
                    +<span x-text="stats.customers.change"></span>
                </span>
            </div>
        </div>

        <div class="stat-card">
            <div class="stat-icon bg-green">
                <svg><use href="#icon-folder"/></svg>
            </div>
            <div class="stat-content">
                <span class="stat-label">Laufende Projekte</span>
                <span class="stat-value" x-text="stats.projects.active">0</span>
            </div>
        </div>

        <div class="stat-card">
            <div class="stat-icon bg-yellow">
                <svg><use href="#icon-euro"/></svg>
            </div>
            <div class="stat-content">
                <span class="stat-label">Offene Rechnungen</span>
                <span class="stat-value" x-text="formatCurrency(stats.invoices.open)">€0</span>
            </div>
        </div>

        <div class="stat-card">
            <div class="stat-icon bg-purple">
                <svg><use href="#icon-clock"/></svg>
            </div>
            <div class="stat-content">
                <span class="stat-label">Erfasste Stunden</span>
                <span class="stat-value" x-text="`${stats.hours.total}h`">0h</span>
                <span class="stat-sub" x-text="`${stats.hours.billable}h abrechenbar`"></span>
            </div>
        </div>
    </section>

    <!-- Team Activity & Upcoming -->
    <section class="dashboard-row">
        <div class="card">
            <div class="card-header">
                <h3>Team-Aktivität</h3>
                <a href="/admin/team/activity" class="link">Alle anzeigen</a>
            </div>
            <div class="card-content">
                <ul class="activity-list">
                    <template x-for="activity in teamActivity" :key="activity.id">
                        <li class="activity-item">
                            <img :src="activity.user.avatar" alt="" class="avatar-sm">
                            <div class="activity-content">
                                <span class="user-name" x-text="activity.user.name"></span>
                                <span x-text="activity.action"></span>
                                <a :href="activity.target_url" x-text="activity.target_name"></a>
                            </div>
                            <time x-text="formatTimeAgo(activity.timestamp)"></time>
                        </li>
                    </template>
                </ul>
            </div>
        </div>

        <div class="card">
            <div class="card-header">
                <h3>Anstehende Termine</h3>
                <a href="/calendar" class="link">Kalender öffnen</a>
            </div>
            <div class="card-content">
                <ul class="upcoming-list">
                    <template x-for="event in upcomingEvents" :key="event.id">
                        <li class="upcoming-item">
                            <div class="event-date">
                                <span class="day" x-text="formatDay(event.date)"></span>
                                <span class="month" x-text="formatMonth(event.date)"></span>
                            </div>
                            <div class="event-content">
                                <span class="event-title" x-text="event.title"></span>
                                <span class="event-time" x-text="event.time"></span>
                                <span class="event-customer" x-text="event.customer_name"></span>
                            </div>
                        </li>
                    </template>
                </ul>
            </div>
        </div>
    </section>

    <!-- Projects Overview -->
    <section class="card">
        <div class="card-header">
            <h3>Projekt-Übersicht</h3>
            <a href="/admin/projects" class="link">Alle Projekte</a>
        </div>
        <div class="card-content">
            <table class="data-table">
                <thead>
                    <tr>
                        <th>Projekt</th>
                        <th>Kunde</th>
                        <th>Bearbeiter</th>
                        <th>Status</th>
                        <th>Fortschritt</th>
                        <th>Fällig</th>
                    </tr>
                </thead>
                <tbody>
                    <template x-for="project in activeProjects" :key="project.id">
                        <tr>
                            <td>
                                <a :href="`/projects/${project.id}`" x-text="project.name"></a>
                            </td>
                            <td x-text="project.customer_name"></td>
                            <td>
                                <div class="avatar-stack">
                                    <template x-for="user in project.assigned_users.slice(0, 3)" :key="user.id">
                                        <img :src="user.avatar" :title="user.name" class="avatar-xs">
                                    </template>
                                    <span x-show="project.assigned_users.length > 3"
                                          x-text="`+${project.assigned_users.length - 3}`"></span>
                                </div>
                            </td>
                            <td>
                                <span class="badge" :class="getProjectStatusBadge(project.status)"
                                      x-text="project.status"></span>
                            </td>
                            <td>
                                <div class="progress-bar">
                                    <div class="progress-fill" :style="`width: ${project.progress}%`"></div>
                                </div>
                                <span class="progress-text" x-text="`${project.progress}%`"></span>
                            </td>
                            <td :class="{ 'text-danger': isOverdue(project.due_date) }"
                                x-text="formatDate(project.due_date)"></td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>
    </section>

    <!-- Revenue Chart -->
    <section class="card">
        <div class="card-header">
            <h3>Umsatzentwicklung</h3>
        </div>
        <div class="card-content">
            <canvas id="revenueChart" height="250"></canvas>
        </div>
    </section>
</div>
```

## Mitarbeiter-Verwaltung

```html
<!-- /templates/orgadmin/team/list.html -->
<div class="page-team" x-data="teamPage()">
    <header class="page-header">
        <h1>Team</h1>
        <div class="header-actions">
            <button @click="openInviteModal" class="btn btn-primary">
                <svg class="icon"><use href="#icon-plus"/></svg>
                Mitarbeiter einladen
            </button>
        </div>
    </header>

    <!-- Team Stats -->
    <div class="team-stats">
        <div class="stat">
            <span class="stat-value" x-text="stats.total"></span>
            <span class="stat-label">Mitarbeiter</span>
        </div>
        <div class="stat">
            <span class="stat-value" x-text="stats.admins"></span>
            <span class="stat-label">Admins</span>
        </div>
        <div class="stat">
            <span class="stat-value" x-text="stats.active_today"></span>
            <span class="stat-label">Heute aktiv</span>
        </div>
        <div class="stat">
            <span class="stat-value" x-text="`${stats.seats_used}/${stats.seats_limit}`"></span>
            <span class="stat-label">Plätze genutzt</span>
        </div>
    </div>

    <!-- Filters -->
    <div class="filters-bar">
        <div class="search-input">
            <svg class="icon"><use href="#icon-search"/></svg>
            <input type="search" placeholder="Mitarbeiter suchen..."
                   x-model="filters.search" @input.debounce.300ms="applyFilters">
        </div>

        <select x-model="filters.role" @change="applyFilters">
            <option value="">Alle Rollen</option>
            <option value="admin">Admin</option>
            <option value="consultant">Berater</option>
            <option value="assistant">Assistent</option>
        </select>

        <select x-model="filters.status" @change="applyFilters">
            <option value="">Alle Status</option>
            <option value="active">Aktiv</option>
            <option value="invited">Eingeladen</option>
            <option value="inactive">Inaktiv</option>
        </select>
    </div>

    <!-- Team Grid -->
    <div class="team-grid">
        <template x-for="member in filteredMembers" :key="member.id">
            <div class="team-card" :class="{ 'inactive': !member.active }">
                <div class="team-card-header">
                    <img :src="member.avatar" alt="" class="avatar-lg">
                    <div class="online-indicator" :class="member.online ? 'online' : 'offline'"></div>
                </div>

                <div class="team-card-body">
                    <h3 x-text="member.name"></h3>
                    <span class="email" x-text="member.email"></span>
                    <span class="badge" :class="getRoleBadge(member.role)"
                          x-text="getRoleLabel(member.role)"></span>
                </div>

                <div class="team-card-stats">
                    <div class="stat">
                        <span class="value" x-text="member.projects_count"></span>
                        <span class="label">Projekte</span>
                    </div>
                    <div class="stat">
                        <span class="value" x-text="`${member.hours_this_month}h`"></span>
                        <span class="label">Stunden/Monat</span>
                    </div>
                </div>

                <div class="team-card-footer">
                    <button @click="editMember(member.id)" class="btn btn-sm btn-secondary">
                        Bearbeiten
                    </button>
                    <div class="dropdown-wrapper" @click.away="member.menuOpen = false">
                        <button @click="member.menuOpen = !member.menuOpen" class="btn-icon">
                            <svg><use href="#icon-dots-vertical"/></svg>
                        </button>
                        <div class="dropdown" x-show="member.menuOpen">
                            <a :href="`/admin/team/${member.id}/activity`">Aktivität</a>
                            <a :href="`/admin/team/${member.id}/timesheet`">Zeiterfassung</a>
                            <hr>
                            <template x-if="member.active">
                                <button @click="deactivateMember(member.id)" class="text-danger">
                                    Deaktivieren
                                </button>
                            </template>
                            <template x-if="!member.active">
                                <button @click="activateMember(member.id)" class="text-success">
                                    Aktivieren
                                </button>
                            </template>
                        </div>
                    </div>
                </div>
            </div>
        </template>

        <!-- Invite Placeholder -->
        <div class="team-card team-card-invite" @click="openInviteModal"
             x-show="stats.seats_used < stats.seats_limit">
            <div class="invite-icon">
                <svg><use href="#icon-user-plus"/></svg>
            </div>
            <span>Mitarbeiter einladen</span>
        </div>
    </div>

    <!-- Invite Modal -->
    <div class="modal" x-show="inviteModalOpen" @click.self="inviteModalOpen = false">
        <div class="modal-content">
            <div class="modal-header">
                <h2>Mitarbeiter einladen</h2>
                <button @click="inviteModalOpen = false" class="btn-icon">×</button>
            </div>
            <form @submit.prevent="sendInvite">
                <div class="modal-body">
                    <div class="form-group">
                        <label>E-Mail-Adresse</label>
                        <input type="email" x-model="inviteForm.email" required
                               placeholder="mitarbeiter@example.de">
                    </div>

                    <div class="form-group">
                        <label>Name</label>
                        <input type="text" x-model="inviteForm.name" required
                               placeholder="Max Mustermann">
                    </div>

                    <div class="form-group">
                        <label>Rolle</label>
                        <select x-model="inviteForm.role" required>
                            <option value="consultant">Berater</option>
                            <option value="assistant">Assistent</option>
                            <option value="admin">Admin</option>
                        </select>
                    </div>

                    <div class="form-group">
                        <label>Teams</label>
                        <div class="checkbox-group">
                            <template x-for="team in teams" :key="team.id">
                                <label class="checkbox-label">
                                    <input type="checkbox" :value="team.id" x-model="inviteForm.teams">
                                    <span x-text="team.name"></span>
                                </label>
                            </template>
                        </div>
                    </div>

                    <div class="form-group">
                        <label class="checkbox-label">
                            <input type="checkbox" x-model="inviteForm.send_welcome_email">
                            <span>Willkommens-E-Mail senden</span>
                        </label>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" @click="inviteModalOpen = false" class="btn btn-secondary">
                        Abbrechen
                    </button>
                    <button type="submit" class="btn btn-primary" :disabled="inviting">
                        <span x-show="!inviting">Einladung senden</span>
                        <span x-show="inviting">Wird gesendet...</span>
                    </button>
                </div>
            </form>
        </div>
    </div>
</div>
```

## Kundenverwaltung

```html
<!-- /templates/orgadmin/customers/list.html -->
<div class="page-customers" x-data="customersPage()">
    <header class="page-header">
        <h1>Kunden</h1>
        <div class="header-actions">
            <button @click="importCustomers" class="btn btn-secondary">
                <svg class="icon"><use href="#icon-upload"/></svg>
                Import
            </button>
            <a href="/admin/customers/new" class="btn btn-primary">
                <svg class="icon"><use href="#icon-plus"/></svg>
                Neuer Kunde
            </a>
        </div>
    </header>

    <!-- Filters -->
    <div class="filters-bar">
        <div class="search-input">
            <svg class="icon"><use href="#icon-search"/></svg>
            <input type="search" placeholder="Kunde suchen..."
                   x-model="filters.search" @input.debounce.300ms="applyFilters">
        </div>

        <select x-model="filters.type" @change="applyFilters">
            <option value="">Alle Typen</option>
            <option value="private">Privat</option>
            <option value="business">Gewerbe</option>
            <option value="public">Öffentlich</option>
        </select>

        <select x-model="filters.status" @change="applyFilters">
            <option value="">Alle Status</option>
            <option value="active">Aktiv</option>
            <option value="inactive">Inaktiv</option>
            <option value="lead">Lead</option>
        </select>

        <select x-model="filters.assigned" @change="applyFilters">
            <option value="">Alle Bearbeiter</option>
            <template x-for="user in teamMembers" :key="user.id">
                <option :value="user.id" x-text="user.name"></option>
            </template>
        </select>
    </div>

    <!-- View Toggle -->
    <div class="view-toggle">
        <button @click="viewMode = 'table'" :class="{ 'active': viewMode === 'table' }">
            <svg><use href="#icon-list"/></svg>
        </button>
        <button @click="viewMode = 'cards'" :class="{ 'active': viewMode === 'cards' }">
            <svg><use href="#icon-grid"/></svg>
        </button>
    </div>

    <!-- Table View -->
    <div class="data-table-container" x-show="viewMode === 'table'">
        <table class="data-table">
            <thead>
                <tr>
                    <th>
                        <input type="checkbox" @change="toggleSelectAll" :checked="allSelected">
                    </th>
                    <th @click="sortBy('name')">
                        Kunde
                        <svg class="sort-icon" x-show="sortColumn === 'name'">
                            <use :href="sortDirection === 'asc' ? '#icon-arrow-up' : '#icon-arrow-down'"/>
                        </svg>
                    </th>
                    <th>Typ</th>
                    <th>Kontakt</th>
                    <th @click="sortBy('projects_count')">Projekte</th>
                    <th @click="sortBy('total_revenue')">Umsatz</th>
                    <th>Bearbeiter</th>
                    <th>Aktionen</th>
                </tr>
            </thead>
            <tbody>
                <template x-for="customer in customers" :key="customer.id">
                    <tr>
                        <td>
                            <input type="checkbox" :value="customer.id" x-model="selected">
                        </td>
                        <td>
                            <div class="customer-cell">
                                <a :href="`/customers/${customer.id}`" class="customer-name"
                                   x-text="customer.name"></a>
                                <span class="customer-company" x-text="customer.company"
                                      x-show="customer.company"></span>
                            </div>
                        </td>
                        <td>
                            <span class="badge" :class="getTypeBadge(customer.type)"
                                  x-text="getTypeLabel(customer.type)"></span>
                        </td>
                        <td>
                            <div class="contact-info">
                                <span x-text="customer.email"></span>
                                <span x-text="customer.phone"></span>
                            </div>
                        </td>
                        <td x-text="customer.projects_count"></td>
                        <td x-text="formatCurrency(customer.total_revenue)"></td>
                        <td>
                            <div class="avatar-stack">
                                <template x-for="user in customer.assigned_users.slice(0, 2)" :key="user.id">
                                    <img :src="user.avatar" :title="user.name" class="avatar-xs">
                                </template>
                            </div>
                        </td>
                        <td>
                            <div class="actions-row">
                                <a :href="`/customers/${customer.id}`" class="btn btn-sm">
                                    Details
                                </a>
                                <a :href="`/projects/new?customer_id=${customer.id}`" class="btn btn-sm btn-primary">
                                    + Projekt
                                </a>
                            </div>
                        </td>
                    </tr>
                </template>
            </tbody>
        </table>
    </div>

    <!-- Card View -->
    <div class="customers-grid" x-show="viewMode === 'cards'">
        <template x-for="customer in customers" :key="customer.id">
            <div class="customer-card">
                <div class="card-header">
                    <h3>
                        <a :href="`/customers/${customer.id}`" x-text="customer.name"></a>
                    </h3>
                    <span class="badge" :class="getTypeBadge(customer.type)"
                          x-text="getTypeLabel(customer.type)"></span>
                </div>
                <div class="card-body">
                    <p class="company" x-text="customer.company" x-show="customer.company"></p>
                    <div class="contact">
                        <a :href="`mailto:${customer.email}`" x-text="customer.email"></a>
                        <span x-text="customer.phone"></span>
                    </div>
                    <div class="address" x-html="formatAddress(customer.address)"></div>
                </div>
                <div class="card-stats">
                    <div class="stat">
                        <span class="value" x-text="customer.projects_count"></span>
                        <span class="label">Projekte</span>
                    </div>
                    <div class="stat">
                        <span class="value" x-text="formatCurrency(customer.total_revenue)"></span>
                        <span class="label">Umsatz</span>
                    </div>
                </div>
                <div class="card-footer">
                    <a :href="`/customers/${customer.id}`" class="btn btn-sm btn-secondary">
                        Details
                    </a>
                    <a :href="`/projects/new?customer_id=${customer.id}`" class="btn btn-sm btn-primary">
                        Neues Projekt
                    </a>
                </div>
            </div>
        </template>
    </div>

    <!-- Pagination -->
    <div class="pagination">
        <span x-text="`${pagination.from}-${pagination.to} von ${pagination.total}`"></span>
        <div class="pagination-controls">
            <button @click="prevPage" :disabled="pagination.page === 1">←</button>
            <span x-text="`Seite ${pagination.page}`"></span>
            <button @click="nextPage" :disabled="pagination.page === pagination.lastPage">→</button>
        </div>
    </div>
</div>
```

## Auftrags-Pipeline

```html
<!-- /templates/orgadmin/orders/pipeline.html -->
<div class="page-pipeline" x-data="pipelinePage()">
    <header class="page-header">
        <h1>Auftrags-Pipeline</h1>
        <div class="header-actions">
            <a href="/admin/orders/new" class="btn btn-primary">
                <svg class="icon"><use href="#icon-plus"/></svg>
                Neuer Auftrag
            </a>
        </div>
    </header>

    <!-- Pipeline Stats -->
    <div class="pipeline-stats">
        <div class="stat">
            <span class="stat-value" x-text="stats.total_count"></span>
            <span class="stat-label">Aufträge gesamt</span>
        </div>
        <div class="stat">
            <span class="stat-value" x-text="formatCurrency(stats.total_value)"></span>
            <span class="stat-label">Gesamtwert</span>
        </div>
        <div class="stat">
            <span class="stat-value" x-text="`${stats.conversion_rate}%`"></span>
            <span class="stat-label">Abschlussrate</span>
        </div>
        <div class="stat">
            <span class="stat-value" x-text="`${stats.avg_days} Tage`"></span>
            <span class="stat-label">Ø Durchlaufzeit</span>
        </div>
    </div>

    <!-- Kanban Board -->
    <div class="kanban-board">
        <template x-for="stage in stages" :key="stage.id">
            <div class="kanban-column"
                 @dragover.prevent
                 @drop="handleDrop($event, stage.id)">
                <div class="column-header" :style="`border-top-color: ${stage.color}`">
                    <h3 x-text="stage.name"></h3>
                    <div class="column-stats">
                        <span x-text="getStageCount(stage.id)"></span>
                        <span x-text="formatCurrency(getStageValue(stage.id))"></span>
                    </div>
                </div>

                <div class="column-content">
                    <template x-for="order in getOrdersByStage(stage.id)" :key="order.id">
                        <div class="kanban-card"
                             draggable="true"
                             @dragstart="handleDragStart($event, order)"
                             @click="openOrderDetail(order.id)">
                            <div class="card-header">
                                <span class="order-number" x-text="order.order_number"></span>
                                <span class="order-type badge badge-sm"
                                      x-text="order.order_type_label"></span>
                            </div>
                            <h4 class="order-title" x-text="order.title"></h4>
                            <div class="order-customer">
                                <svg class="icon-sm"><use href="#icon-user"/></svg>
                                <span x-text="order.customer_name"></span>
                            </div>
                            <div class="order-value" x-text="formatCurrency(order.value)"></div>
                            <div class="card-footer">
                                <div class="assigned-user" x-show="order.assigned_user">
                                    <img :src="order.assigned_user?.avatar" class="avatar-xs">
                                    <span x-text="order.assigned_user?.name"></span>
                                </div>
                                <span class="due-date" :class="{ 'overdue': isOverdue(order.due_date) }"
                                      x-text="formatDate(order.due_date)"></span>
                            </div>
                        </div>
                    </template>

                    <!-- Add Order Button -->
                    <button class="add-order-btn" @click="createOrderInStage(stage.id)">
                        <svg><use href="#icon-plus"/></svg>
                        Auftrag hinzufügen
                    </button>
                </div>
            </div>
        </template>
    </div>
</div>
```

## Organisations-Einstellungen

```html
<!-- /templates/orgadmin/settings/organization.html -->
<div class="page-settings" x-data="orgSettingsPage()">
    <header class="page-header">
        <h1>Organisations-Einstellungen</h1>
    </header>

    <div class="settings-layout">
        <!-- Settings Navigation -->
        <nav class="settings-nav">
            <a href="/admin/settings/organization" class="active">Organisation</a>
            <a href="/admin/settings/branding">Branding</a>
            <a href="/admin/settings/email">E-Mail</a>
            <a href="/admin/settings/templates">Vorlagen</a>
            <a href="/admin/settings/integrations">Integrationen</a>
            <a href="/admin/settings/billing">Abrechnung</a>
        </nav>

        <!-- Settings Content -->
        <div class="settings-content">
            <form @submit.prevent="saveSettings">
                <!-- Basic Info -->
                <section class="settings-section">
                    <h2>Grunddaten</h2>

                    <div class="form-group">
                        <label>Firmenname</label>
                        <input type="text" x-model="settings.name" required>
                    </div>

                    <div class="form-row">
                        <div class="form-group">
                            <label>Rechtsform</label>
                            <select x-model="settings.legal_form">
                                <option value="einzelunternehmen">Einzelunternehmen</option>
                                <option value="gbr">GbR</option>
                                <option value="gmbh">GmbH</option>
                                <option value="ug">UG (haftungsbeschränkt)</option>
                                <option value="ohg">OHG</option>
                                <option value="kg">KG</option>
                            </select>
                        </div>

                        <div class="form-group">
                            <label>Steuernummer</label>
                            <input type="text" x-model="settings.tax_number"
                                   placeholder="12/345/67890">
                        </div>
                    </div>

                    <div class="form-group">
                        <label>USt-IdNr.</label>
                        <input type="text" x-model="settings.vat_id" placeholder="DE123456789">
                    </div>
                </section>

                <!-- Address -->
                <section class="settings-section">
                    <h2>Adresse</h2>

                    <div class="form-group">
                        <label>Straße & Hausnummer</label>
                        <input type="text" x-model="settings.address.street">
                    </div>

                    <div class="form-row">
                        <div class="form-group" style="flex: 1">
                            <label>PLZ</label>
                            <input type="text" x-model="settings.address.zip">
                        </div>
                        <div class="form-group" style="flex: 3">
                            <label>Ort</label>
                            <input type="text" x-model="settings.address.city">
                        </div>
                    </div>

                    <div class="form-group">
                        <label>Land</label>
                        <select x-model="settings.address.country">
                            <option value="DE">Deutschland</option>
                            <option value="AT">Österreich</option>
                            <option value="CH">Schweiz</option>
                        </select>
                    </div>
                </section>

                <!-- Contact -->
                <section class="settings-section">
                    <h2>Kontakt</h2>

                    <div class="form-group">
                        <label>E-Mail</label>
                        <input type="email" x-model="settings.email">
                    </div>

                    <div class="form-group">
                        <label>Telefon</label>
                        <input type="tel" x-model="settings.phone">
                    </div>

                    <div class="form-group">
                        <label>Website</label>
                        <input type="url" x-model="settings.website">
                    </div>
                </section>

                <!-- Certifications -->
                <section class="settings-section">
                    <h2>Zertifizierungen</h2>

                    <div class="certifications-list">
                        <template x-for="cert in settings.certifications" :key="cert.id">
                            <div class="certification-item">
                                <div class="cert-info">
                                    <span class="cert-name" x-text="cert.name"></span>
                                    <span class="cert-issuer" x-text="cert.issuer"></span>
                                    <span class="cert-expiry">
                                        Gültig bis: <span x-text="formatDate(cert.valid_until)"></span>
                                    </span>
                                </div>
                                <div class="cert-actions">
                                    <button type="button" @click="editCertification(cert.id)"
                                            class="btn btn-sm">Bearbeiten</button>
                                    <button type="button" @click="removeCertification(cert.id)"
                                            class="btn btn-sm btn-danger">Entfernen</button>
                                </div>
                            </div>
                        </template>
                    </div>

                    <button type="button" @click="addCertification" class="btn btn-secondary">
                        + Zertifizierung hinzufügen
                    </button>
                </section>

                <!-- BAFA/dena Registration -->
                <section class="settings-section">
                    <h2>BAFA / dena Registrierung</h2>

                    <div class="form-group">
                        <label>BAFA Beraternummer</label>
                        <input type="text" x-model="settings.bafa_number"
                               placeholder="123456">
                    </div>

                    <div class="form-group">
                        <label>dena Expertenliste ID</label>
                        <input type="text" x-model="settings.dena_id"
                               placeholder="EXP-12345">
                    </div>

                    <div class="form-group-checkbox">
                        <input type="checkbox" id="bafa_verified" x-model="settings.bafa_verified">
                        <label for="bafa_verified">BAFA-Verifizierung abgeschlossen</label>
                    </div>
                </section>

                <div class="form-actions">
                    <button type="submit" class="btn btn-primary" :disabled="saving">
                        <span x-show="!saving">Speichern</span>
                        <span x-show="saving">Wird gespeichert...</span>
                    </button>
                </div>
            </form>
        </div>
    </div>
</div>
```

## Branding-Einstellungen

```html
<!-- /templates/orgadmin/settings/branding.html -->
<div class="page-settings" x-data="brandingSettingsPage()">
    <header class="page-header">
        <h1>Branding</h1>
    </header>

    <div class="settings-layout">
        <nav class="settings-nav">
            <a href="/admin/settings/organization">Organisation</a>
            <a href="/admin/settings/branding" class="active">Branding</a>
            <a href="/admin/settings/email">E-Mail</a>
            <a href="/admin/settings/templates">Vorlagen</a>
            <a href="/admin/settings/integrations">Integrationen</a>
            <a href="/admin/settings/billing">Abrechnung</a>
        </nav>

        <div class="settings-content">
            <form @submit.prevent="saveSettings">
                <!-- Logo -->
                <section class="settings-section">
                    <h2>Logo</h2>

                    <div class="form-group">
                        <label>Hauptlogo</label>
                        <div class="logo-upload">
                            <div class="logo-preview">
                                <img :src="settings.logo || '/static/img/placeholder-logo.svg'"
                                     alt="Logo">
                            </div>
                            <div class="upload-actions">
                                <input type="file" id="logo-upload" accept="image/*"
                                       @change="uploadLogo" hidden>
                                <label for="logo-upload" class="btn btn-secondary">
                                    Logo hochladen
                                </label>
                                <button type="button" @click="removeLogo" class="btn btn-danger"
                                        x-show="settings.logo">
                                    Entfernen
                                </button>
                            </div>
                            <p class="help-text">Empfohlen: SVG oder PNG, max. 500KB</p>
                        </div>
                    </div>

                    <div class="form-group">
                        <label>Logo für dunklen Hintergrund</label>
                        <div class="logo-upload dark-bg">
                            <div class="logo-preview">
                                <img :src="settings.logo_dark || settings.logo || '/static/img/placeholder-logo.svg'"
                                     alt="Logo (dunkel)">
                            </div>
                            <div class="upload-actions">
                                <input type="file" id="logo-dark-upload" accept="image/*"
                                       @change="uploadLogoDark" hidden>
                                <label for="logo-dark-upload" class="btn btn-secondary">
                                    Logo hochladen
                                </label>
                            </div>
                        </div>
                    </div>
                </section>

                <!-- Colors -->
                <section class="settings-section">
                    <h2>Farben</h2>

                    <div class="color-pickers">
                        <div class="form-group">
                            <label>Primärfarbe</label>
                            <div class="color-input">
                                <input type="color" x-model="settings.colors.primary">
                                <input type="text" x-model="settings.colors.primary"
                                       placeholder="#3B82F6">
                            </div>
                        </div>

                        <div class="form-group">
                            <label>Sekundärfarbe</label>
                            <div class="color-input">
                                <input type="color" x-model="settings.colors.secondary">
                                <input type="text" x-model="settings.colors.secondary"
                                       placeholder="#10B981">
                            </div>
                        </div>

                        <div class="form-group">
                            <label>Akzentfarbe</label>
                            <div class="color-input">
                                <input type="color" x-model="settings.colors.accent">
                                <input type="text" x-model="settings.colors.accent"
                                       placeholder="#F59E0B">
                            </div>
                        </div>
                    </div>

                    <div class="color-preview">
                        <h4>Vorschau</h4>
                        <div class="preview-buttons">
                            <button type="button" class="btn"
                                    :style="`background: ${settings.colors.primary}; color: white`">
                                Primär
                            </button>
                            <button type="button" class="btn"
                                    :style="`background: ${settings.colors.secondary}; color: white`">
                                Sekundär
                            </button>
                            <button type="button" class="btn"
                                    :style="`background: ${settings.colors.accent}; color: white`">
                                Akzent
                            </button>
                        </div>
                    </div>
                </section>

                <!-- Document Templates -->
                <section class="settings-section">
                    <h2>Dokument-Vorlagen</h2>

                    <div class="form-group">
                        <label>Briefkopf</label>
                        <div class="template-upload">
                            <img :src="settings.letterhead" alt="" class="template-preview"
                                 x-show="settings.letterhead">
                            <input type="file" id="letterhead-upload" accept="image/*"
                                   @change="uploadLetterhead" hidden>
                            <label for="letterhead-upload" class="btn btn-secondary">
                                Briefkopf hochladen
                            </label>
                        </div>
                    </div>

                    <div class="form-group">
                        <label>Fußzeile für Dokumente</label>
                        <textarea x-model="settings.document_footer" rows="3"
                                  placeholder="Bankverbindung, Kontaktdaten etc."></textarea>
                    </div>

                    <div class="form-group">
                        <label>Signatur-Bild</label>
                        <div class="template-upload">
                            <img :src="settings.signature" alt="" class="signature-preview"
                                 x-show="settings.signature">
                            <input type="file" id="signature-upload" accept="image/*"
                                   @change="uploadSignature" hidden>
                            <label for="signature-upload" class="btn btn-secondary">
                                Signatur hochladen
                            </label>
                        </div>
                    </div>
                </section>

                <div class="form-actions">
                    <button type="submit" class="btn btn-primary" :disabled="saving">
                        Speichern
                    </button>
                </div>
            </form>
        </div>
    </div>
</div>
```

## Abrechnung & Plan

```html
<!-- /templates/orgadmin/settings/billing.html -->
<div class="page-settings" x-data="billingSettingsPage()">
    <header class="page-header">
        <h1>Abrechnung</h1>
    </header>

    <div class="settings-layout">
        <nav class="settings-nav">
            <a href="/admin/settings/organization">Organisation</a>
            <a href="/admin/settings/branding">Branding</a>
            <a href="/admin/settings/email">E-Mail</a>
            <a href="/admin/settings/templates">Vorlagen</a>
            <a href="/admin/settings/integrations">Integrationen</a>
            <a href="/admin/settings/billing" class="active">Abrechnung</a>
        </nav>

        <div class="settings-content">
            <!-- Current Plan -->
            <section class="settings-section">
                <h2>Aktueller Plan</h2>

                <div class="plan-card current">
                    <div class="plan-header">
                        <h3 x-text="currentPlan.name"></h3>
                        <span class="plan-price">
                            <span x-text="formatCurrency(currentPlan.price)"></span>/Monat
                        </span>
                    </div>
                    <div class="plan-features">
                        <ul>
                            <li>
                                <svg class="icon-check"><use href="#icon-check"/></svg>
                                <span x-text="`${currentPlan.limits.users} Benutzer`"></span>
                            </li>
                            <li>
                                <svg class="icon-check"><use href="#icon-check"/></svg>
                                <span x-text="`${currentPlan.limits.projects} Projekte`"></span>
                            </li>
                            <li>
                                <svg class="icon-check"><use href="#icon-check"/></svg>
                                <span x-text="`${currentPlan.limits.storage_gb} GB Speicher`"></span>
                            </li>
                        </ul>
                    </div>
                    <div class="plan-usage">
                        <h4>Aktuelle Nutzung</h4>
                        <div class="usage-item">
                            <span>Benutzer</span>
                            <div class="usage-bar">
                                <div class="usage-fill"
                                     :style="`width: ${(usage.users / currentPlan.limits.users) * 100}%`"></div>
                            </div>
                            <span x-text="`${usage.users}/${currentPlan.limits.users}`"></span>
                        </div>
                        <div class="usage-item">
                            <span>Projekte</span>
                            <div class="usage-bar">
                                <div class="usage-fill"
                                     :style="`width: ${(usage.projects / currentPlan.limits.projects) * 100}%`"></div>
                            </div>
                            <span x-text="`${usage.projects}/${currentPlan.limits.projects}`"></span>
                        </div>
                        <div class="usage-item">
                            <span>Speicher</span>
                            <div class="usage-bar">
                                <div class="usage-fill"
                                     :style="`width: ${(usage.storage_gb / currentPlan.limits.storage_gb) * 100}%`"></div>
                            </div>
                            <span x-text="`${usage.storage_gb}/${currentPlan.limits.storage_gb} GB`"></span>
                        </div>
                    </div>
                    <button @click="openChangePlanModal" class="btn btn-primary">
                        Plan ändern
                    </button>
                </div>
            </section>

            <!-- Active Modules -->
            <section class="settings-section">
                <h2>Aktive Module</h2>

                <div class="modules-list">
                    <template x-for="module in activeModules" :key="module.id">
                        <div class="module-item">
                            <div class="module-info">
                                <svg class="module-icon"><use :href="`#icon-${module.icon}`"/></svg>
                                <div>
                                    <span class="module-name" x-text="module.name"></span>
                                    <span class="module-price"
                                          x-text="module.included ? 'Im Plan enthalten' : formatCurrency(module.price) + '/Monat'"></span>
                                </div>
                            </div>
                            <button @click="toggleModule(module.id)"
                                    class="btn btn-sm"
                                    :class="module.active ? 'btn-danger' : 'btn-success'"
                                    x-text="module.active ? 'Deaktivieren' : 'Aktivieren'"
                                    :disabled="module.included">
                            </button>
                        </div>
                    </template>
                </div>

                <a href="/admin/settings/modules" class="btn btn-secondary">
                    Alle Module ansehen
                </a>
            </section>

            <!-- Payment Method -->
            <section class="settings-section">
                <h2>Zahlungsmethode</h2>

                <div class="payment-method" x-show="paymentMethod">
                    <div class="card-info">
                        <svg class="card-brand"><use :href="`#icon-${paymentMethod?.brand}`"/></svg>
                        <span x-text="`•••• •••• •••• ${paymentMethod?.last4}`"></span>
                        <span x-text="`Gültig bis ${paymentMethod?.exp_month}/${paymentMethod?.exp_year}`"></span>
                    </div>
                    <button @click="updatePaymentMethod" class="btn btn-secondary">
                        Ändern
                    </button>
                </div>

                <div class="no-payment" x-show="!paymentMethod">
                    <p>Keine Zahlungsmethode hinterlegt</p>
                    <button @click="addPaymentMethod" class="btn btn-primary">
                        Zahlungsmethode hinzufügen
                    </button>
                </div>
            </section>

            <!-- Billing History -->
            <section class="settings-section">
                <h2>Rechnungshistorie</h2>

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
                        <template x-for="invoice in invoices" :key="invoice.id">
                            <tr>
                                <td x-text="invoice.number"></td>
                                <td x-text="formatDate(invoice.date)"></td>
                                <td x-text="formatCurrency(invoice.amount)"></td>
                                <td>
                                    <span class="badge"
                                          :class="invoice.paid ? 'badge-success' : 'badge-warning'"
                                          x-text="invoice.paid ? 'Bezahlt' : 'Offen'"></span>
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
            </section>

            <!-- Billing Address -->
            <section class="settings-section">
                <h2>Rechnungsadresse</h2>

                <form @submit.prevent="saveBillingAddress">
                    <div class="form-group">
                        <label>Firmenname</label>
                        <input type="text" x-model="billingAddress.company">
                    </div>
                    <div class="form-group">
                        <label>Straße & Hausnummer</label>
                        <input type="text" x-model="billingAddress.street">
                    </div>
                    <div class="form-row">
                        <div class="form-group">
                            <label>PLZ</label>
                            <input type="text" x-model="billingAddress.zip">
                        </div>
                        <div class="form-group">
                            <label>Ort</label>
                            <input type="text" x-model="billingAddress.city">
                        </div>
                    </div>
                    <div class="form-group">
                        <label>Land</label>
                        <select x-model="billingAddress.country">
                            <option value="DE">Deutschland</option>
                            <option value="AT">Österreich</option>
                            <option value="CH">Schweiz</option>
                        </select>
                    </div>
                    <button type="submit" class="btn btn-primary">Speichern</button>
                </form>
            </section>
        </div>
    </div>
</div>
```

## JavaScript Controllers

```javascript
// /static/js/orgadmin/dashboard.js

function orgadminDashboard() {
    return {
        dateRange: '30d',
        stats: {
            customers: { active: 0, change: 0 },
            projects: { active: 0 },
            invoices: { open: 0 },
            hours: { total: 0, billable: 0 }
        },
        teamActivity: [],
        upcomingEvents: [],
        activeProjects: [],

        async init() {
            await this.loadData();
            this.initCharts();
        },

        async loadData() {
            const [stats, activity, events, projects] = await Promise.all([
                fetch(`/api/v1/admin/dashboard/stats?range=${this.dateRange}`).then(r => r.json()),
                fetch('/api/v1/admin/dashboard/activity?limit=5').then(r => r.json()),
                fetch('/api/v1/admin/dashboard/events?limit=5').then(r => r.json()),
                fetch('/api/v1/admin/dashboard/projects?limit=5').then(r => r.json())
            ]);

            this.stats = stats;
            this.teamActivity = activity;
            this.upcomingEvents = events;
            this.activeProjects = projects;
        },

        initCharts() {
            const ctx = document.getElementById('revenueChart').getContext('2d');
            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Umsatz',
                        data: [],
                        borderColor: '#3b82f6',
                        backgroundColor: 'rgba(59, 130, 246, 0.1)',
                        fill: true,
                        tension: 0.4
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: { display: false }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: (value) => this.formatCurrency(value)
                            }
                        }
                    }
                }
            });

            this.loadChartData();
        },

        async loadChartData() {
            const response = await fetch(`/api/v1/admin/dashboard/revenue?range=${this.dateRange}`);
            const data = await response.json();
            // Update chart...
        },

        formatCurrency(amount) {
            return new Intl.NumberFormat('de-DE', {
                style: 'currency',
                currency: 'EUR'
            }).format(amount);
        },

        formatTimeAgo(timestamp) {
            const rtf = new Intl.RelativeTimeFormat('de', { numeric: 'auto' });
            const diff = Date.now() - new Date(timestamp).getTime();
            const minutes = Math.floor(diff / 60000);

            if (minutes < 60) return rtf.format(-minutes, 'minute');
            const hours = Math.floor(minutes / 60);
            if (hours < 24) return rtf.format(-hours, 'hour');
            const days = Math.floor(hours / 24);
            return rtf.format(-days, 'day');
        },

        formatDate(dateStr) {
            return new Date(dateStr).toLocaleDateString('de-DE');
        },

        formatDay(dateStr) {
            return new Date(dateStr).getDate();
        },

        formatMonth(dateStr) {
            return new Date(dateStr).toLocaleDateString('de-DE', { month: 'short' });
        },

        getProjectStatusBadge(status) {
            const badges = {
                'in_progress': 'badge-info',
                'review': 'badge-warning',
                'completed': 'badge-success',
                'on_hold': 'badge-secondary'
            };
            return badges[status] || 'badge-secondary';
        },

        isOverdue(dateStr) {
            return new Date(dateStr) < new Date();
        }
    };
}

function teamPage() {
    return {
        members: [],
        stats: {
            total: 0,
            admins: 0,
            active_today: 0,
            seats_used: 0,
            seats_limit: 10
        },
        filters: {
            search: '',
            role: '',
            status: ''
        },
        inviteModalOpen: false,
        inviteForm: {
            email: '',
            name: '',
            role: 'consultant',
            teams: [],
            send_welcome_email: true
        },
        teams: [],
        inviting: false,

        get filteredMembers() {
            return this.members.filter(member => {
                if (this.filters.search) {
                    const search = this.filters.search.toLowerCase();
                    if (!member.name.toLowerCase().includes(search) &&
                        !member.email.toLowerCase().includes(search)) {
                        return false;
                    }
                }
                if (this.filters.role && member.role !== this.filters.role) return false;
                if (this.filters.status) {
                    if (this.filters.status === 'active' && !member.active) return false;
                    if (this.filters.status === 'inactive' && member.active) return false;
                }
                return true;
            });
        },

        async init() {
            await Promise.all([
                this.loadMembers(),
                this.loadTeams()
            ]);
        },

        async loadMembers() {
            const response = await fetch('/api/v1/admin/team');
            const data = await response.json();
            this.members = data.members;
            this.stats = data.stats;
        },

        async loadTeams() {
            const response = await fetch('/api/v1/admin/teams');
            this.teams = await response.json();
        },

        openInviteModal() {
            this.inviteForm = {
                email: '',
                name: '',
                role: 'consultant',
                teams: [],
                send_welcome_email: true
            };
            this.inviteModalOpen = true;
        },

        async sendInvite() {
            this.inviting = true;
            try {
                const response = await fetch('/api/v1/admin/team/invite', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(this.inviteForm)
                });

                if (response.ok) {
                    this.inviteModalOpen = false;
                    await this.loadMembers();
                    window.dispatchEvent(new CustomEvent('notification', {
                        detail: { message: 'Einladung gesendet', type: 'success' }
                    }));
                }
            } finally {
                this.inviting = false;
            }
        },

        getRoleBadge(role) {
            const badges = {
                admin: 'badge-danger',
                consultant: 'badge-info',
                assistant: 'badge-secondary'
            };
            return badges[role] || 'badge-secondary';
        },

        getRoleLabel(role) {
            const labels = {
                admin: 'Admin',
                consultant: 'Berater',
                assistant: 'Assistent'
            };
            return labels[role] || role;
        }
    };
}
```

## API-Endpunkte

```python
# /app/api/v1/orgadmin/routes.py

from fastapi import APIRouter, Depends, HTTPException
from app.core.auth import require_org_admin, get_current_organization
from app.services.organization import OrganizationService

router = APIRouter(prefix="/admin", tags=["org-admin"])

@router.get("/dashboard/stats")
async def get_dashboard_stats(
    range: str = "30d",
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Dashboard-Statistiken für Organisation"""
    return await service.get_dashboard_stats(org.id, range)


@router.get("/team")
async def list_team_members(
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Listet alle Teammitglieder der Organisation"""
    return await service.list_team_members(org.id)


@router.post("/team/invite")
async def invite_team_member(
    data: InviteUserRequest,
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Lädt neuen Mitarbeiter ein"""
    return await service.invite_user(
        org_id=org.id,
        inviter_id=current_user.id,
        data=data
    )


@router.get("/customers")
async def list_customers(
    search: str = None,
    type: str = None,
    status: str = None,
    page: int = 1,
    per_page: int = 25,
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Listet Kunden der Organisation"""
    return await service.list_customers(
        org_id=org.id,
        search=search,
        customer_type=type,
        status=status,
        page=page,
        per_page=per_page
    )


@router.get("/settings")
async def get_organization_settings(
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Holt Organisations-Einstellungen"""
    return await service.get_settings(org.id)


@router.put("/settings")
async def update_organization_settings(
    data: UpdateOrganizationSettings,
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Aktualisiert Organisations-Einstellungen"""
    return await service.update_settings(
        org_id=org.id,
        admin_id=current_user.id,
        data=data
    )


@router.get("/billing")
async def get_billing_info(
    current_user = Depends(require_org_admin),
    org = Depends(get_current_organization),
    service: OrganizationService = Depends()
):
    """Holt Abrechnungsinformationen"""
    return await service.get_billing_info(org.id)
```

## Zusammenfassung

Die Org-Admin-Oberfläche bietet:

1. **Team-Verwaltung**: Mitarbeiter einladen, Rollen zuweisen, Aktivität überwachen
2. **Kundenverwaltung**: CRM für die eigene Organisation
3. **Projekt-Übersicht**: Alle Projekte der Organisation
4. **Auftrags-Pipeline**: Kanban-Board für Auftragsverwaltung
5. **Einstellungen**: Organisation, Branding, E-Mail, Templates
6. **Abrechnung**: Plan-Übersicht, Module, Rechnungen

**Wichtige Einschränkungen:**
- Kein Zugriff auf andere Organisationen
- Keine Systemkonfiguration
- Keine Plattform-weiten Einstellungen
- Module können nur im Rahmen des Plans genutzt werden
