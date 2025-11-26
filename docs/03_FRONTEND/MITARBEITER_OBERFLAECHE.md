# Mitarbeiter-Oberfläche

## Übersicht

Die Mitarbeiter-Oberfläche ist die Hauptarbeitsumgebung für Energieberater und Assistenten. Sie fokussiert auf 0% Bürokratie - automatische Zeiterfassung, geführte Workflows und intelligente Vorlagen.

## Berechtigungsebene

```
┌────────────────────────────────────────────────────────────────────────────┐
│                         MITARBEITER ZUGRIFF                                 │
├────────────────────────────────────────────────────────────────────────────┤
│  ✓ Eigene zugewiesene Projekte                                             │
│  ✓ Eigene zugewiesene Kunden                                               │
│  ✓ Eigene Zeiterfassung (automatisch)                                      │
│  ✓ Dokumente erstellen und bearbeiten                                      │
│  ✓ Berichte generieren (eigene Projekte)                                   │
│  ✓ KI-Assistent nutzen                                                     │
│  ✓ Eigenes Profil bearbeiten                                               │
│  ✗ Andere Mitarbeiter verwalten                                            │
│  ✗ Organisations-Einstellungen                                             │
│  ✗ Abrechnung und Finanzen                                                 │
│  ✗ Nicht zugewiesene Projekte/Kunden                                       │
└────────────────────────────────────────────────────────────────────────────┘
```

## Haupt-Navigation

```html
<!-- /templates/app/layout.html -->
<div class="app-layout" x-data="appLayout()">
    <!-- Sidebar -->
    <aside class="sidebar" :class="{ 'collapsed': sidebarCollapsed }">
        <div class="sidebar-header">
            <img :src="organization.logo || '/static/img/logo.svg'" alt="" class="logo">
            <button @click="sidebarCollapsed = !sidebarCollapsed" class="btn-collapse">
                <svg class="icon"><use href="#icon-chevron-left"/></svg>
            </button>
        </div>

        <nav class="sidebar-nav">
            <!-- Quick Actions -->
            <div class="quick-actions">
                <button @click="openQuickAction('project')" class="quick-btn" title="Neues Projekt">
                    <svg><use href="#icon-folder-plus"/></svg>
                </button>
                <button @click="openQuickAction('document')" class="quick-btn" title="Neues Dokument">
                    <svg><use href="#icon-file-plus"/></svg>
                </button>
                <button @click="openQuickAction('time')" class="quick-btn" title="Zeit erfassen">
                    <svg><use href="#icon-clock"/></svg>
                </button>
            </div>

            <!-- Main Nav -->
            <a href="/dashboard" class="nav-item" :class="{ 'active': currentPage === 'dashboard' }">
                <svg class="icon"><use href="#icon-home"/></svg>
                <span>Start</span>
            </a>

            <a href="/projects" class="nav-item" :class="{ 'active': currentPage.startsWith('project') }">
                <svg class="icon"><use href="#icon-folder"/></svg>
                <span>Meine Projekte</span>
                <span class="badge" x-show="counts.active_projects" x-text="counts.active_projects"></span>
            </a>

            <a href="/customers" class="nav-item" :class="{ 'active': currentPage.startsWith('customer') }">
                <svg class="icon"><use href="#icon-users"/></svg>
                <span>Meine Kunden</span>
            </a>

            <a href="/documents" class="nav-item" :class="{ 'active': currentPage.startsWith('document') }">
                <svg class="icon"><use href="#icon-file-text"/></svg>
                <span>Dokumente</span>
            </a>

            <a href="/calculations" class="nav-item" :class="{ 'active': currentPage.startsWith('calc') }">
                <svg class="icon"><use href="#icon-calculator"/></svg>
                <span>Berechnungen</span>
            </a>

            <a href="/calendar" class="nav-item" :class="{ 'active': currentPage === 'calendar' }">
                <svg class="icon"><use href="#icon-calendar"/></svg>
                <span>Kalender</span>
            </a>

            <a href="/tasks" class="nav-item" :class="{ 'active': currentPage === 'tasks' }">
                <svg class="icon"><use href="#icon-check-square"/></svg>
                <span>Aufgaben</span>
                <span class="badge badge-warning" x-show="counts.pending_tasks" x-text="counts.pending_tasks"></span>
            </a>

            <!-- Divider -->
            <hr class="nav-divider">

            <a href="/ai-assistant" class="nav-item" :class="{ 'active': currentPage === 'ai' }">
                <svg class="icon"><use href="#icon-sparkles"/></svg>
                <span>KI-Assistent</span>
            </a>

            <a href="/knowledge" class="nav-item" :class="{ 'active': currentPage === 'knowledge' }">
                <svg class="icon"><use href="#icon-book"/></svg>
                <span>Wissensdatenbank</span>
            </a>
        </nav>

        <!-- Time Tracking Widget -->
        <div class="sidebar-footer">
            <div class="time-widget" x-data="timeWidget()">
                <div class="time-display">
                    <span class="time-label">Heute erfasst</span>
                    <span class="time-value" x-text="formatDuration(todayTime)"></span>
                </div>
                <div class="current-task" x-show="activeTask">
                    <div class="task-indicator running"></div>
                    <span class="task-name" x-text="activeTask?.project_name"></span>
                    <span class="task-time" x-text="formatDuration(activeTask?.duration)"></span>
                </div>
                <div class="time-actions">
                    <button @click="pauseTracking" x-show="activeTask" class="btn btn-sm">
                        Pause
                    </button>
                    <button @click="openTimeEntry" class="btn btn-sm btn-secondary">
                        Manuell
                    </button>
                </div>
            </div>
        </div>
    </aside>

    <!-- Main Content -->
    <main class="main-content">
        <header class="main-header">
            <div class="header-left">
                <div class="breadcrumbs" x-html="breadcrumbs"></div>
            </div>
            <div class="header-right">
                <!-- Global Search -->
                <div class="search-global" x-data="globalSearch()">
                    <div class="search-trigger" @click="open = true">
                        <svg class="icon"><use href="#icon-search"/></svg>
                        <span>Suchen...</span>
                        <kbd>⌘K</kbd>
                    </div>

                    <!-- Search Modal -->
                    <div class="search-modal" x-show="open" @click.self="open = false"
                         @keydown.escape="open = false">
                        <div class="search-content">
                            <input type="search" x-model="query"
                                   @input.debounce.200ms="search"
                                   placeholder="Projekte, Kunden, Dokumente suchen..."
                                   x-ref="searchInput">
                            <div class="search-results" x-show="results.length > 0">
                                <template x-for="group in groupedResults" :key="group.type">
                                    <div class="result-group">
                                        <h4 x-text="group.label"></h4>
                                        <template x-for="item in group.items" :key="item.id">
                                            <a :href="item.url" class="result-item"
                                               @click="open = false">
                                                <svg class="icon"><use :href="`#icon-${item.icon}`"/></svg>
                                                <div class="result-content">
                                                    <span class="result-title" x-text="item.title"></span>
                                                    <span class="result-sub" x-text="item.subtitle"></span>
                                                </div>
                                            </a>
                                        </template>
                                    </div>
                                </template>
                            </div>
                            <div class="search-empty" x-show="query && results.length === 0">
                                Keine Ergebnisse für "<span x-text="query"></span>"
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Notifications -->
                <div class="notifications" x-data="notifications()">
                    <button @click="togglePanel" class="btn-icon">
                        <svg class="icon"><use href="#icon-bell"/></svg>
                        <span class="badge" x-show="unreadCount > 0" x-text="unreadCount"></span>
                    </button>

                    <div class="notifications-panel" x-show="panelOpen" @click.away="panelOpen = false">
                        <div class="panel-header">
                            <h3>Benachrichtigungen</h3>
                            <button @click="markAllRead" class="btn-link">Alle gelesen</button>
                        </div>
                        <div class="notifications-list">
                            <template x-for="notification in items" :key="notification.id">
                                <div class="notification-item" :class="{ 'unread': !notification.read }">
                                    <svg class="icon"><use :href="`#icon-${notification.icon}`"/></svg>
                                    <div class="notification-content">
                                        <p x-html="notification.message"></p>
                                        <time x-text="formatTimeAgo(notification.created_at)"></time>
                                    </div>
                                </div>
                            </template>
                        </div>
                    </div>
                </div>

                <!-- User Menu -->
                <div class="user-menu" x-data="{ open: false }" @click.away="open = false">
                    <button @click="open = !open" class="user-button">
                        <img :src="user.avatar" alt="" class="avatar">
                        <span x-text="user.name" class="user-name"></span>
                    </button>
                    <div class="dropdown" x-show="open">
                        <a href="/profile">Mein Profil</a>
                        <a href="/settings">Einstellungen</a>
                        <a href="/help">Hilfe</a>
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

    <!-- AI Assistant Sidebar (toggleable) -->
    <aside class="ai-sidebar" x-show="aiSidebarOpen" x-transition>
        <div class="ai-header">
            <h3>KI-Assistent</h3>
            <button @click="aiSidebarOpen = false" class="btn-icon">×</button>
        </div>
        <div class="ai-content" x-data="aiAssistant()">
            <!-- AI Chat Interface -->
        </div>
    </aside>
</div>
```

## Dashboard

```html
<!-- /templates/app/dashboard.html -->
<div class="dashboard" x-data="dashboard()">
    <!-- Welcome Section -->
    <section class="welcome-section">
        <div class="welcome-content">
            <h1>Guten <span x-text="getGreeting()"></span>, <span x-text="user.first_name"></span>!</h1>
            <p x-text="getTodaySummary()"></p>
        </div>
        <div class="quick-stats">
            <div class="stat">
                <span class="value" x-text="stats.projects_active"></span>
                <span class="label">Aktive Projekte</span>
            </div>
            <div class="stat">
                <span class="value" x-text="stats.tasks_due_today"></span>
                <span class="label">Aufgaben heute</span>
            </div>
            <div class="stat">
                <span class="value" x-text="`${stats.hours_today}h`"></span>
                <span class="label">Erfasste Zeit</span>
            </div>
        </div>
    </section>

    <!-- Today's Focus -->
    <section class="today-focus">
        <h2>Heute anstehend</h2>
        <div class="focus-items">
            <template x-for="item in todayItems" :key="item.id">
                <div class="focus-item" :class="`focus-${item.type}`">
                    <div class="focus-time" x-text="item.time"></div>
                    <div class="focus-content">
                        <span class="focus-type badge" :class="getTypeBadge(item.type)"
                              x-text="getTypeLabel(item.type)"></span>
                        <h4>
                            <a :href="item.url" x-text="item.title"></a>
                        </h4>
                        <p x-text="item.description"></p>
                    </div>
                    <div class="focus-actions">
                        <button @click="startWork(item)" class="btn btn-sm btn-primary"
                                x-show="item.type === 'project' || item.type === 'task'">
                            Starten
                        </button>
                    </div>
                </div>
            </template>

            <div class="focus-empty" x-show="todayItems.length === 0">
                <svg><use href="#icon-check-circle"/></svg>
                <p>Keine Termine oder Aufgaben für heute!</p>
            </div>
        </div>
    </section>

    <!-- Active Projects -->
    <section class="active-projects">
        <div class="section-header">
            <h2>Meine aktiven Projekte</h2>
            <a href="/projects" class="link">Alle anzeigen</a>
        </div>

        <div class="projects-grid">
            <template x-for="project in activeProjects" :key="project.id">
                <div class="project-card" @click="window.location.href = `/projects/${project.id}`">
                    <div class="project-header">
                        <span class="project-type badge" :class="getProjectTypeBadge(project.type)"
                              x-text="project.type_label"></span>
                        <span class="project-status" :class="project.status" x-text="project.status_label"></span>
                    </div>

                    <h3 class="project-name" x-text="project.name"></h3>
                    <p class="project-customer" x-text="project.customer_name"></p>

                    <div class="project-progress">
                        <div class="progress-bar">
                            <div class="progress-fill" :style="`width: ${project.progress}%`"></div>
                        </div>
                        <span x-text="`${project.progress}%`"></span>
                    </div>

                    <div class="project-meta">
                        <div class="meta-item">
                            <svg class="icon-sm"><use href="#icon-calendar"/></svg>
                            <span x-text="formatDate(project.due_date)"></span>
                        </div>
                        <div class="meta-item">
                            <svg class="icon-sm"><use href="#icon-clock"/></svg>
                            <span x-text="`${project.hours_spent}h / ${project.hours_estimated}h`"></span>
                        </div>
                    </div>

                    <div class="project-tasks">
                        <span x-text="`${project.tasks_completed}/${project.tasks_total} Aufgaben`"></span>
                    </div>
                </div>
            </template>

            <!-- New Project Card -->
            <div class="project-card project-card-new" @click="openNewProject">
                <div class="new-project-content">
                    <svg><use href="#icon-plus"/></svg>
                    <span>Neues Projekt</span>
                </div>
            </div>
        </div>
    </section>

    <!-- Recent Documents & Activity -->
    <div class="dashboard-row">
        <section class="recent-documents">
            <div class="section-header">
                <h2>Letzte Dokumente</h2>
                <a href="/documents" class="link">Alle anzeigen</a>
            </div>
            <ul class="document-list">
                <template x-for="doc in recentDocuments" :key="doc.id">
                    <li class="document-item">
                        <svg class="icon"><use :href="`#icon-file-${doc.type}`"/></svg>
                        <div class="document-info">
                            <a :href="`/documents/${doc.id}`" x-text="doc.name"></a>
                            <span class="document-meta">
                                <span x-text="doc.project_name"></span> •
                                <span x-text="formatTimeAgo(doc.updated_at)"></span>
                            </span>
                        </div>
                        <button @click.stop="openDocument(doc.id)" class="btn-icon">
                            <svg><use href="#icon-external-link"/></svg>
                        </button>
                    </li>
                </template>
            </ul>
        </section>

        <section class="recent-activity">
            <div class="section-header">
                <h2>Meine Aktivität</h2>
            </div>
            <ul class="activity-timeline">
                <template x-for="activity in recentActivity" :key="activity.id">
                    <li class="activity-item">
                        <div class="activity-dot" :class="`dot-${activity.type}`"></div>
                        <div class="activity-content">
                            <p x-html="activity.description"></p>
                            <time x-text="formatTimeAgo(activity.timestamp)"></time>
                        </div>
                    </li>
                </template>
            </ul>
        </section>
    </div>
</div>
```

## Projekt-Ansicht

```html
<!-- /templates/app/project/detail.html -->
<div class="project-detail" x-data="projectDetail()">
    <!-- Project Header -->
    <header class="project-header">
        <div class="header-main">
            <div class="project-title">
                <span class="badge" :class="getProjectTypeBadge(project.type)"
                      x-text="project.type_label"></span>
                <h1 x-text="project.name"></h1>
            </div>
            <div class="project-customer">
                <a :href="`/customers/${project.customer_id}`" x-text="project.customer_name"></a>
            </div>
        </div>

        <div class="header-actions">
            <button @click="toggleTimeTracking" class="btn"
                    :class="isTracking ? 'btn-danger' : 'btn-success'">
                <svg class="icon"><use :href="isTracking ? '#icon-pause' : '#icon-play'"/></svg>
                <span x-text="isTracking ? 'Stoppen' : 'Zeit erfassen'"></span>
            </button>
            <button @click="openAIAssistant" class="btn btn-secondary">
                <svg class="icon"><use href="#icon-sparkles"/></svg>
                KI-Hilfe
            </button>
            <div class="dropdown-wrapper">
                <button @click="actionsOpen = !actionsOpen" class="btn btn-secondary">
                    Aktionen ▾
                </button>
                <div class="dropdown" x-show="actionsOpen" @click.away="actionsOpen = false">
                    <a href="#" @click.prevent="duplicateProject">Duplizieren</a>
                    <a href="#" @click.prevent="exportProject">Exportieren</a>
                    <a href="#" @click.prevent="archiveProject">Archivieren</a>
                </div>
            </div>
        </div>
    </header>

    <!-- Progress Bar -->
    <div class="project-progress-bar">
        <div class="progress-info">
            <span>Fortschritt</span>
            <span x-text="`${project.progress}%`"></span>
        </div>
        <div class="progress-track">
            <div class="progress-fill" :style="`width: ${project.progress}%`"></div>
        </div>
    </div>

    <!-- Tab Navigation -->
    <nav class="project-tabs">
        <button @click="activeTab = 'overview'"
                :class="{ 'active': activeTab === 'overview' }">Übersicht</button>
        <button @click="activeTab = 'building'"
                :class="{ 'active': activeTab === 'building' }">Gebäude</button>
        <button @click="activeTab = 'calculations'"
                :class="{ 'active': activeTab === 'calculations' }">Berechnungen</button>
        <button @click="activeTab = 'documents'"
                :class="{ 'active': activeTab === 'documents' }">Dokumente</button>
        <button @click="activeTab = 'tasks'"
                :class="{ 'active': activeTab === 'tasks' }">Aufgaben</button>
        <button @click="activeTab = 'time'"
                :class="{ 'active': activeTab === 'time' }">Zeiterfassung</button>
        <button @click="activeTab = 'notes'"
                :class="{ 'active': activeTab === 'notes' }">Notizen</button>
    </nav>

    <!-- Tab: Overview -->
    <div class="tab-content" x-show="activeTab === 'overview'">
        <div class="overview-grid">
            <!-- Project Info -->
            <div class="info-card">
                <h3>Projektdetails</h3>
                <dl class="info-list">
                    <dt>Projekt-Nr.</dt>
                    <dd x-text="project.project_number"></dd>

                    <dt>Typ</dt>
                    <dd x-text="project.type_label"></dd>

                    <dt>Status</dt>
                    <dd>
                        <select x-model="project.status" @change="updateStatus" class="status-select">
                            <option value="planning">Planung</option>
                            <option value="data_collection">Datenerfassung</option>
                            <option value="calculation">Berechnung</option>
                            <option value="report_writing">Berichterstellung</option>
                            <option value="review">Prüfung</option>
                            <option value="completed">Abgeschlossen</option>
                        </select>
                    </dd>

                    <dt>Erstellt</dt>
                    <dd x-text="formatDate(project.created_at)"></dd>

                    <dt>Fällig</dt>
                    <dd :class="{ 'text-danger': isOverdue(project.due_date) }"
                        x-text="formatDate(project.due_date)"></dd>
                </dl>
            </div>

            <!-- Building Summary -->
            <div class="info-card">
                <h3>Gebäude</h3>
                <dl class="info-list" x-show="project.building">
                    <dt>Adresse</dt>
                    <dd x-html="formatAddress(project.building?.address)"></dd>

                    <dt>Gebäudetyp</dt>
                    <dd x-text="project.building?.building_type_label"></dd>

                    <dt>Baujahr</dt>
                    <dd x-text="project.building?.construction_year"></dd>

                    <dt>Wohnfläche</dt>
                    <dd x-text="`${project.building?.living_area} m²`"></dd>

                    <dt>Energieeffizienzklasse</dt>
                    <dd>
                        <span class="efficiency-badge" :class="`efficiency-${project.building?.efficiency_class?.toLowerCase()}`"
                              x-text="project.building?.efficiency_class || 'N/A'"></span>
                    </dd>
                </dl>
                <p x-show="!project.building" class="text-muted">
                    Gebäudedaten noch nicht erfasst
                </p>
                <a :href="`/projects/${project.id}/building`" class="btn btn-sm btn-secondary">
                    Gebäude bearbeiten
                </a>
            </div>

            <!-- Quick Tasks -->
            <div class="info-card">
                <h3>Offene Aufgaben</h3>
                <ul class="task-list-compact">
                    <template x-for="task in openTasks.slice(0, 5)" :key="task.id">
                        <li class="task-item" @click="toggleTask(task.id)">
                            <input type="checkbox" :checked="task.completed">
                            <span x-text="task.title"></span>
                            <span class="task-due" x-show="task.due_date"
                                  x-text="formatDate(task.due_date)"></span>
                        </li>
                    </template>
                </ul>
                <button @click="activeTab = 'tasks'" class="btn btn-sm btn-link">
                    Alle Aufgaben (<span x-text="openTasks.length"></span>)
                </button>
            </div>

            <!-- Time Summary -->
            <div class="info-card">
                <h3>Zeiterfassung</h3>
                <div class="time-summary">
                    <div class="time-stat">
                        <span class="time-value" x-text="`${project.hours_spent}h`"></span>
                        <span class="time-label">Erfasst</span>
                    </div>
                    <div class="time-stat">
                        <span class="time-value" x-text="`${project.hours_estimated}h`"></span>
                        <span class="time-label">Geschätzt</span>
                    </div>
                    <div class="time-stat">
                        <span class="time-value"
                              :class="{ 'text-danger': project.hours_spent > project.hours_estimated }"
                              x-text="`${project.hours_estimated - project.hours_spent}h`"></span>
                        <span class="time-label">Verbleibend</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- Guided Workflow -->
        <div class="workflow-guide" x-show="project.workflow_enabled">
            <h3>Nächste Schritte</h3>
            <div class="workflow-steps">
                <template x-for="(step, index) in workflowSteps" :key="step.id">
                    <div class="workflow-step"
                         :class="{ 'completed': step.completed, 'current': step.current }">
                        <div class="step-indicator">
                            <span class="step-number" x-text="index + 1"></span>
                        </div>
                        <div class="step-content">
                            <h4 x-text="step.title"></h4>
                            <p x-text="step.description"></p>
                            <a :href="step.action_url" class="btn btn-sm btn-primary"
                               x-show="step.current">
                                <span x-text="step.action_label"></span>
                            </a>
                        </div>
                    </div>
                </template>
            </div>
        </div>
    </div>

    <!-- Tab: Building Data -->
    <div class="tab-content" x-show="activeTab === 'building'">
        <div class="building-editor" x-data="buildingEditor()">
            <!-- Building data forms -->
            <div class="editor-sections">
                <section class="editor-section">
                    <h3>Allgemeine Daten</h3>
                    <div class="form-grid">
                        <div class="form-group">
                            <label>Gebäudetyp</label>
                            <select x-model="building.type">
                                <option value="single_family">Einfamilienhaus</option>
                                <option value="multi_family">Mehrfamilienhaus</option>
                                <option value="apartment">Wohnung</option>
                                <option value="commercial">Gewerbe</option>
                                <option value="mixed">Gemischt</option>
                            </select>
                        </div>

                        <div class="form-group">
                            <label>Baujahr</label>
                            <input type="number" x-model="building.construction_year"
                                   min="1800" max="2025">
                        </div>

                        <div class="form-group">
                            <label>Letzte Sanierung</label>
                            <input type="number" x-model="building.last_renovation_year"
                                   min="1900" max="2025">
                        </div>

                        <div class="form-group">
                            <label>Wohnfläche (m²)</label>
                            <input type="number" x-model="building.living_area" step="0.1">
                        </div>

                        <div class="form-group">
                            <label>Nutzfläche (m²)</label>
                            <input type="number" x-model="building.usable_area" step="0.1">
                        </div>

                        <div class="form-group">
                            <label>Geschosse</label>
                            <input type="number" x-model="building.floors" min="1" max="100">
                        </div>
                    </div>
                </section>

                <section class="editor-section">
                    <h3>Gebäudehülle</h3>
                    <!-- Envelope components: walls, roof, windows, etc. -->
                    <div class="envelope-components">
                        <template x-for="component in building.envelope" :key="component.id">
                            <div class="envelope-card">
                                <div class="envelope-header">
                                    <span x-text="component.type_label"></span>
                                    <button @click="editEnvelopeComponent(component.id)" class="btn-icon">
                                        <svg><use href="#icon-edit"/></svg>
                                    </button>
                                </div>
                                <dl>
                                    <dt>Fläche</dt>
                                    <dd x-text="`${component.area} m²`"></dd>
                                    <dt>U-Wert</dt>
                                    <dd x-text="`${component.u_value} W/(m²K)`"></dd>
                                </dl>
                            </div>
                        </template>
                        <button @click="addEnvelopeComponent" class="envelope-card envelope-add">
                            <svg><use href="#icon-plus"/></svg>
                            <span>Bauteil hinzufügen</span>
                        </button>
                    </div>
                </section>

                <section class="editor-section">
                    <h3>Anlagentechnik</h3>
                    <!-- HVAC systems -->
                    <div class="systems-list">
                        <template x-for="system in building.systems" :key="system.id">
                            <div class="system-card">
                                <svg class="system-icon"><use :href="`#icon-${system.type}`"/></svg>
                                <div class="system-info">
                                    <span class="system-type" x-text="system.type_label"></span>
                                    <span class="system-details" x-text="system.details"></span>
                                </div>
                                <button @click="editSystem(system.id)" class="btn btn-sm">
                                    Bearbeiten
                                </button>
                            </div>
                        </template>
                        <button @click="addSystem" class="btn btn-secondary">
                            + Anlage hinzufügen
                        </button>
                    </div>
                </section>
            </div>

            <div class="editor-actions">
                <button @click="saveBuilding" class="btn btn-primary" :disabled="saving">
                    Speichern
                </button>
                <button @click="runCalculation" class="btn btn-secondary">
                    Berechnung starten
                </button>
            </div>
        </div>
    </div>

    <!-- Tab: Calculations -->
    <div class="tab-content" x-show="activeTab === 'calculations'">
        <div class="calculations-panel" x-data="calculationsPanel()">
            <!-- Available Calculations -->
            <div class="calc-types">
                <h3>Verfügbare Berechnungen</h3>
                <div class="calc-grid">
                    <div class="calc-card" @click="startCalculation('energy_demand')">
                        <svg><use href="#icon-bolt"/></svg>
                        <h4>Energiebedarf</h4>
                        <p>DIN V 18599 konform</p>
                    </div>
                    <div class="calc-card" @click="startCalculation('heat_load')">
                        <svg><use href="#icon-thermometer"/></svg>
                        <h4>Heizlast</h4>
                        <p>DIN EN 12831</p>
                    </div>
                    <div class="calc-card" @click="startCalculation('co2')">
                        <svg><use href="#icon-leaf"/></svg>
                        <h4>CO₂-Bilanz</h4>
                        <p>Emissionsbewertung</p>
                    </div>
                    <div class="calc-card" @click="startCalculation('economic')">
                        <svg><use href="#icon-euro"/></svg>
                        <h4>Wirtschaftlichkeit</h4>
                        <p>Kosten-Nutzen-Analyse</p>
                    </div>
                </div>
            </div>

            <!-- Calculation History -->
            <div class="calc-history">
                <h3>Durchgeführte Berechnungen</h3>
                <table class="data-table">
                    <thead>
                        <tr>
                            <th>Typ</th>
                            <th>Datum</th>
                            <th>Status</th>
                            <th>Ergebnis</th>
                            <th>Aktionen</th>
                        </tr>
                    </thead>
                    <tbody>
                        <template x-for="calc in calculations" :key="calc.id">
                            <tr>
                                <td x-text="calc.type_label"></td>
                                <td x-text="formatDateTime(calc.created_at)"></td>
                                <td>
                                    <span class="badge" :class="getCalcStatusBadge(calc.status)"
                                          x-text="calc.status_label"></span>
                                </td>
                                <td x-text="calc.result_summary || '-'"></td>
                                <td>
                                    <button @click="viewCalculation(calc.id)" class="btn btn-sm">
                                        Details
                                    </button>
                                    <button @click="useInReport(calc.id)" class="btn btn-sm btn-secondary">
                                        In Bericht
                                    </button>
                                </td>
                            </tr>
                        </template>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <!-- Tab: Documents -->
    <div class="tab-content" x-show="activeTab === 'documents'">
        <div class="documents-panel" x-data="documentsPanel()">
            <div class="documents-header">
                <div class="view-toggle">
                    <button @click="viewMode = 'grid'" :class="{ 'active': viewMode === 'grid' }">
                        <svg><use href="#icon-grid"/></svg>
                    </button>
                    <button @click="viewMode = 'list'" :class="{ 'active': viewMode === 'list' }">
                        <svg><use href="#icon-list"/></svg>
                    </button>
                </div>
                <div class="documents-actions">
                    <button @click="uploadDocument" class="btn btn-secondary">
                        <svg class="icon"><use href="#icon-upload"/></svg>
                        Hochladen
                    </button>
                    <button @click="createDocument" class="btn btn-primary">
                        <svg class="icon"><use href="#icon-plus"/></svg>
                        Neues Dokument
                    </button>
                </div>
            </div>

            <!-- Document Grid -->
            <div class="documents-grid" x-show="viewMode === 'grid'">
                <template x-for="doc in documents" :key="doc.id">
                    <div class="document-card" @click="openDocument(doc.id)">
                        <div class="document-preview">
                            <img :src="doc.thumbnail" alt="" x-show="doc.thumbnail">
                            <svg x-show="!doc.thumbnail"><use :href="`#icon-file-${doc.type}`"/></svg>
                        </div>
                        <div class="document-info">
                            <h4 x-text="doc.name"></h4>
                            <span class="document-meta" x-text="formatTimeAgo(doc.updated_at)"></span>
                        </div>
                    </div>
                </template>

                <!-- New Document Templates -->
                <div class="document-card document-new" @click="openTemplateSelector">
                    <div class="document-preview">
                        <svg><use href="#icon-file-plus"/></svg>
                    </div>
                    <div class="document-info">
                        <h4>Aus Vorlage erstellen</h4>
                    </div>
                </div>
            </div>

            <!-- Template Selector Modal -->
            <div class="modal" x-show="templateSelectorOpen" @click.self="templateSelectorOpen = false">
                <div class="modal-content modal-lg">
                    <div class="modal-header">
                        <h2>Dokument aus Vorlage erstellen</h2>
                        <button @click="templateSelectorOpen = false" class="btn-icon">×</button>
                    </div>
                    <div class="modal-body">
                        <div class="template-categories">
                            <button @click="templateCategory = 'reports'"
                                    :class="{ 'active': templateCategory === 'reports' }">
                                Berichte
                            </button>
                            <button @click="templateCategory = 'certificates'"
                                    :class="{ 'active': templateCategory === 'certificates' }">
                                Ausweise
                            </button>
                            <button @click="templateCategory = 'letters'"
                                    :class="{ 'active': templateCategory === 'letters' }">
                                Anschreiben
                            </button>
                            <button @click="templateCategory = 'forms'"
                                    :class="{ 'active': templateCategory === 'forms' }">
                                Formulare
                            </button>
                        </div>

                        <div class="template-grid">
                            <template x-for="template in filteredTemplates" :key="template.id">
                                <div class="template-card" @click="selectTemplate(template.id)">
                                    <img :src="template.preview" alt="" class="template-preview">
                                    <div class="template-info">
                                        <h4 x-text="template.name"></h4>
                                        <p x-text="template.description"></p>
                                    </div>
                                </div>
                            </template>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Tab: Tasks -->
    <div class="tab-content" x-show="activeTab === 'tasks'">
        <div class="tasks-panel" x-data="tasksPanel()">
            <div class="tasks-header">
                <div class="tasks-filters">
                    <button @click="taskFilter = 'all'" :class="{ 'active': taskFilter === 'all' }">
                        Alle
                    </button>
                    <button @click="taskFilter = 'open'" :class="{ 'active': taskFilter === 'open' }">
                        Offen
                    </button>
                    <button @click="taskFilter = 'completed'" :class="{ 'active': taskFilter === 'completed' }">
                        Erledigt
                    </button>
                </div>
                <button @click="addTask" class="btn btn-primary">
                    + Aufgabe
                </button>
            </div>

            <ul class="tasks-list">
                <template x-for="task in filteredTasks" :key="task.id">
                    <li class="task-item" :class="{ 'completed': task.completed }">
                        <input type="checkbox" :checked="task.completed"
                               @change="toggleTask(task.id)">
                        <div class="task-content" @click="editTask(task.id)">
                            <span class="task-title" x-text="task.title"></span>
                            <div class="task-meta">
                                <span class="task-due" x-show="task.due_date"
                                      :class="{ 'overdue': isOverdue(task.due_date) }"
                                      x-text="formatDate(task.due_date)"></span>
                                <span class="task-priority badge" :class="`badge-${task.priority}`"
                                      x-text="task.priority_label"></span>
                            </div>
                        </div>
                        <button @click="deleteTask(task.id)" class="btn-icon text-danger">
                            <svg><use href="#icon-trash"/></svg>
                        </button>
                    </li>
                </template>
            </ul>

            <!-- Quick Add -->
            <div class="task-quick-add">
                <input type="text" placeholder="Neue Aufgabe hinzufügen..."
                       x-model="newTaskTitle"
                       @keydown.enter="quickAddTask">
                <button @click="quickAddTask" class="btn btn-primary">+</button>
            </div>
        </div>
    </div>

    <!-- Tab: Time Tracking -->
    <div class="tab-content" x-show="activeTab === 'time'">
        <div class="time-panel" x-data="timePanel()">
            <div class="time-summary-cards">
                <div class="time-card">
                    <span class="time-value" x-text="`${totals.this_week}h`"></span>
                    <span class="time-label">Diese Woche</span>
                </div>
                <div class="time-card">
                    <span class="time-value" x-text="`${totals.this_month}h`"></span>
                    <span class="time-label">Dieser Monat</span>
                </div>
                <div class="time-card">
                    <span class="time-value" x-text="`${totals.total}h`"></span>
                    <span class="time-label">Gesamt</span>
                </div>
            </div>

            <div class="time-entries">
                <div class="entries-header">
                    <h3>Zeiteinträge</h3>
                    <button @click="addTimeEntry" class="btn btn-secondary">
                        + Manueller Eintrag
                    </button>
                </div>

                <template x-for="(entries, date) in groupedEntries" :key="date">
                    <div class="entries-day">
                        <h4 class="day-header">
                            <span x-text="formatDateLabel(date)"></span>
                            <span class="day-total" x-text="`${getDayTotal(date)}h`"></span>
                        </h4>
                        <ul class="entries-list">
                            <template x-for="entry in entries" :key="entry.id">
                                <li class="entry-item">
                                    <div class="entry-time">
                                        <span x-text="formatTime(entry.start_time)"></span>
                                        <span>-</span>
                                        <span x-text="formatTime(entry.end_time)"></span>
                                    </div>
                                    <div class="entry-duration" x-text="`${entry.duration}h`"></div>
                                    <div class="entry-description" x-text="entry.description || 'Keine Beschreibung'"></div>
                                    <div class="entry-actions">
                                        <button @click="editEntry(entry.id)" class="btn-icon">
                                            <svg><use href="#icon-edit"/></svg>
                                        </button>
                                        <button @click="deleteEntry(entry.id)" class="btn-icon text-danger">
                                            <svg><use href="#icon-trash"/></svg>
                                        </button>
                                    </div>
                                </li>
                            </template>
                        </ul>
                    </div>
                </template>
            </div>
        </div>
    </div>
</div>
```

## Automatische Zeiterfassung

```javascript
// /static/js/app/auto-timetracking.js

/**
 * Automatische Zeiterfassung ohne Benutzerinteraktion
 * Erfasst Aktivität basierend auf Fokus, Mausbewegung und Tastatureingaben
 */
class AutoTimeTracker {
    constructor() {
        this.isTracking = false;
        this.currentSession = null;
        this.lastActivity = Date.now();
        this.idleTimeout = 5 * 60 * 1000; // 5 Minuten Inaktivität
        this.syncInterval = 60 * 1000; // Sync alle 60 Sekunden

        this.init();
    }

    init() {
        // Activity listeners
        document.addEventListener('mousemove', () => this.recordActivity());
        document.addEventListener('keydown', () => this.recordActivity());
        document.addEventListener('click', () => this.recordActivity());
        document.addEventListener('scroll', () => this.recordActivity());

        // Visibility change (Tab wechsel)
        document.addEventListener('visibilitychange', () => {
            if (document.hidden) {
                this.pauseTracking('tab_hidden');
            } else {
                this.resumeTracking();
            }
        });

        // Page context detection
        this.detectContext();

        // Start sync interval
        setInterval(() => this.syncToServer(), this.syncInterval);

        // Idle check
        setInterval(() => this.checkIdle(), 30000);

        // Unload handler
        window.addEventListener('beforeunload', () => this.saveOnUnload());
    }

    detectContext() {
        // Detect current page/project context
        const url = window.location.pathname;
        const projectMatch = url.match(/\/projects\/(\d+)/);
        const customerMatch = url.match(/\/customers\/(\d+)/);
        const documentMatch = url.match(/\/documents\/(\d+)/);

        this.context = {
            url: url,
            project_id: projectMatch ? parseInt(projectMatch[1]) : null,
            customer_id: customerMatch ? parseInt(customerMatch[1]) : null,
            document_id: documentMatch ? parseInt(documentMatch[1]) : null,
            page_type: this.getPageType(url)
        };

        // Auto-start tracking if in project context
        if (this.context.project_id && !this.isTracking) {
            this.startTracking();
        }
    }

    getPageType(url) {
        if (url.includes('/projects/')) return 'project';
        if (url.includes('/customers/')) return 'customer';
        if (url.includes('/documents/')) return 'document';
        if (url.includes('/calculations')) return 'calculation';
        if (url.includes('/dashboard')) return 'dashboard';
        return 'other';
    }

    recordActivity() {
        this.lastActivity = Date.now();

        if (!this.isTracking && this.context?.project_id) {
            this.startTracking();
        }
    }

    startTracking() {
        if (this.isTracking) return;

        this.isTracking = true;
        this.currentSession = {
            id: this.generateSessionId(),
            project_id: this.context.project_id,
            start_time: new Date().toISOString(),
            activities: [],
            context: { ...this.context }
        };

        this.logActivity('session_start');
        this.broadcastState('started');
    }

    pauseTracking(reason = 'manual') {
        if (!this.isTracking) return;

        this.logActivity('pause', { reason });
        this.isTracking = false;
        this.syncToServer();
        this.broadcastState('paused');
    }

    resumeTracking() {
        if (this.currentSession && !this.isTracking) {
            this.isTracking = true;
            this.logActivity('resume');
            this.broadcastState('resumed');
        }
    }

    stopTracking() {
        if (!this.currentSession) return;

        this.logActivity('session_end');
        this.currentSession.end_time = new Date().toISOString();
        this.syncToServer(true); // Force sync
        this.isTracking = false;
        this.currentSession = null;
        this.broadcastState('stopped');
    }

    logActivity(type, data = {}) {
        if (!this.currentSession) return;

        this.currentSession.activities.push({
            timestamp: new Date().toISOString(),
            type: type,
            url: window.location.pathname,
            ...data
        });
    }

    checkIdle() {
        if (!this.isTracking) return;

        const idleTime = Date.now() - this.lastActivity;

        if (idleTime > this.idleTimeout) {
            this.pauseTracking('idle');
        }
    }

    async syncToServer(force = false) {
        if (!this.currentSession) return;
        if (!force && this.currentSession.activities.length < 5) return;

        try {
            const response = await fetch('/api/v1/time-tracking/sync', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    session: this.currentSession,
                    force: force
                })
            });

            if (response.ok) {
                // Clear synced activities
                this.currentSession.activities = [];
            }
        } catch (error) {
            // Store locally for later sync
            this.storeLocally();
        }
    }

    storeLocally() {
        const stored = JSON.parse(localStorage.getItem('timetracking_pending') || '[]');
        stored.push(this.currentSession);
        localStorage.setItem('timetracking_pending', JSON.stringify(stored));
    }

    saveOnUnload() {
        if (this.currentSession) {
            this.currentSession.end_time = new Date().toISOString();
            // Synchronous storage for unload
            navigator.sendBeacon('/api/v1/time-tracking/sync', JSON.stringify({
                session: this.currentSession
            }));
        }
    }

    broadcastState(state) {
        // Broadcast to other tabs via BroadcastChannel
        const channel = new BroadcastChannel('timetracking');
        channel.postMessage({
            type: 'state_change',
            state: state,
            session: this.currentSession
        });
    }

    generateSessionId() {
        return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    }

    // Public API
    getCurrentSession() {
        return this.currentSession;
    }

    getTodayTotal() {
        // Fetch from API or local cache
        return this.todayTotal || 0;
    }
}

// Initialize on page load
const autoTimeTracker = new AutoTimeTracker();

// Expose for manual control
window.timeTracker = autoTimeTracker;
```

## KI-Assistent Integration

```html
<!-- /templates/app/partials/ai-assistant.html -->
<div class="ai-assistant" x-data="aiAssistant()">
    <!-- Chat Messages -->
    <div class="ai-messages" x-ref="messagesContainer">
        <template x-for="message in messages" :key="message.id">
            <div class="ai-message" :class="`message-${message.role}`">
                <div class="message-avatar">
                    <img :src="message.role === 'user' ? user.avatar : '/static/img/ai-avatar.svg'" alt="">
                </div>
                <div class="message-content">
                    <div class="message-text" x-html="formatMessage(message.content)"></div>

                    <!-- Tool Results -->
                    <div class="tool-results" x-show="message.tool_results?.length > 0">
                        <template x-for="result in message.tool_results" :key="result.id">
                            <div class="tool-result" :class="`tool-${result.type}`">
                                <div class="tool-header">
                                    <svg class="icon"><use :href="`#icon-${result.icon}`"/></svg>
                                    <span x-text="result.title"></span>
                                </div>
                                <div class="tool-content" x-html="result.content"></div>
                            </div>
                        </template>
                    </div>

                    <time x-text="formatTime(message.timestamp)"></time>
                </div>
            </div>
        </template>

        <!-- Typing Indicator -->
        <div class="ai-message message-assistant" x-show="isTyping">
            <div class="message-avatar">
                <img src="/static/img/ai-avatar.svg" alt="">
            </div>
            <div class="message-content">
                <div class="typing-indicator">
                    <span></span><span></span><span></span>
                </div>
            </div>
        </div>
    </div>

    <!-- Suggested Actions -->
    <div class="ai-suggestions" x-show="suggestions.length > 0">
        <template x-for="suggestion in suggestions" :key="suggestion.id">
            <button @click="useSuggestion(suggestion)" class="suggestion-btn">
                <svg class="icon"><use :href="`#icon-${suggestion.icon}`"/></svg>
                <span x-text="suggestion.label"></span>
            </button>
        </template>
    </div>

    <!-- Input Area -->
    <div class="ai-input">
        <div class="input-context" x-show="currentContext">
            <span class="context-label">Kontext:</span>
            <span class="context-value" x-text="currentContext"></span>
            <button @click="clearContext" class="btn-icon-sm">×</button>
        </div>

        <div class="input-row">
            <button @click="attachFile" class="btn-icon" title="Datei anhängen">
                <svg><use href="#icon-paperclip"/></svg>
            </button>
            <textarea x-model="inputMessage"
                      @keydown.enter.prevent="sendMessage"
                      placeholder="Frag mich etwas..."
                      rows="1"
                      x-ref="input"></textarea>
            <button @click="sendMessage" class="btn btn-primary" :disabled="!inputMessage.trim() || isTyping">
                <svg><use href="#icon-send"/></svg>
            </button>
        </div>
    </div>
</div>

<script>
function aiAssistant() {
    return {
        messages: [],
        inputMessage: '',
        isTyping: false,
        suggestions: [],
        currentContext: null,

        async init() {
            // Load conversation history
            await this.loadHistory();

            // Set context based on current page
            this.detectContext();

            // Generate initial suggestions
            this.generateSuggestions();
        },

        detectContext() {
            const url = window.location.pathname;

            if (url.includes('/projects/')) {
                const projectId = url.match(/\/projects\/(\d+)/)?.[1];
                if (projectId) {
                    this.currentContext = `Projekt: ${window.projectData?.name || projectId}`;
                }
            }
        },

        async sendMessage() {
            if (!this.inputMessage.trim() || this.isTyping) return;

            const userMessage = {
                id: Date.now(),
                role: 'user',
                content: this.inputMessage,
                timestamp: new Date().toISOString()
            };

            this.messages.push(userMessage);
            this.inputMessage = '';
            this.isTyping = true;
            this.scrollToBottom();

            try {
                const response = await fetch('/api/v1/ai/chat', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        message: userMessage.content,
                        context: this.getContext(),
                        history: this.messages.slice(-10)
                    })
                });

                const data = await response.json();

                this.messages.push({
                    id: Date.now(),
                    role: 'assistant',
                    content: data.response,
                    tool_results: data.tool_results,
                    timestamp: new Date().toISOString()
                });

                // Update suggestions
                this.suggestions = data.suggestions || [];

            } catch (error) {
                this.messages.push({
                    id: Date.now(),
                    role: 'assistant',
                    content: 'Es tut mir leid, es ist ein Fehler aufgetreten. Bitte versuche es erneut.',
                    timestamp: new Date().toISOString()
                });
            } finally {
                this.isTyping = false;
                this.scrollToBottom();
            }
        },

        getContext() {
            return {
                page: window.location.pathname,
                project_id: window.projectData?.id,
                customer_id: window.customerData?.id,
                document_id: window.documentData?.id
            };
        },

        generateSuggestions() {
            const context = this.getContext();

            if (context.project_id) {
                this.suggestions = [
                    { id: 1, icon: 'calculator', label: 'Energiebedarf berechnen' },
                    { id: 2, icon: 'file-text', label: 'Bericht erstellen' },
                    { id: 3, icon: 'lightbulb', label: 'Sanierungsvorschläge' }
                ];
            } else {
                this.suggestions = [
                    { id: 1, icon: 'folder-plus', label: 'Neues Projekt anlegen' },
                    { id: 2, icon: 'search', label: 'Projekt suchen' },
                    { id: 3, icon: 'book', label: 'Norm nachschlagen' }
                ];
            }
        },

        useSuggestion(suggestion) {
            this.inputMessage = suggestion.label;
            this.sendMessage();
        },

        formatMessage(content) {
            // Convert markdown to HTML
            return marked.parse(content);
        },

        formatTime(timestamp) {
            return new Date(timestamp).toLocaleTimeString('de-DE', {
                hour: '2-digit',
                minute: '2-digit'
            });
        },

        scrollToBottom() {
            this.$nextTick(() => {
                const container = this.$refs.messagesContainer;
                container.scrollTop = container.scrollHeight;
            });
        },

        clearContext() {
            this.currentContext = null;
        },

        async loadHistory() {
            try {
                const response = await fetch('/api/v1/ai/history?limit=20');
                const data = await response.json();
                this.messages = data.messages || [];
            } catch (error) {
                console.error('Failed to load chat history:', error);
            }
        }
    };
}
</script>
```

## API-Endpunkte

```python
# /app/api/v1/app/routes.py

from fastapi import APIRouter, Depends, HTTPException
from app.core.auth import get_current_user
from app.services.project import ProjectService
from app.services.timetracking import TimeTrackingService
from app.services.ai import AIService

router = APIRouter(prefix="/app", tags=["app"])

@router.get("/dashboard")
async def get_dashboard(
    current_user = Depends(get_current_user),
    project_service: ProjectService = Depends(),
    time_service: TimeTrackingService = Depends()
):
    """Dashboard-Daten für Mitarbeiter"""
    return {
        "stats": await project_service.get_user_stats(current_user.id),
        "active_projects": await project_service.get_user_projects(
            current_user.id, status="active", limit=6
        ),
        "today_items": await project_service.get_today_items(current_user.id),
        "recent_documents": await project_service.get_recent_documents(
            current_user.id, limit=5
        ),
        "recent_activity": await project_service.get_user_activity(
            current_user.id, limit=10
        )
    }


@router.get("/projects")
async def list_my_projects(
    status: str = None,
    search: str = None,
    page: int = 1,
    per_page: int = 20,
    current_user = Depends(get_current_user),
    service: ProjectService = Depends()
):
    """Listet nur zugewiesene Projekte des Benutzers"""
    return await service.get_user_projects(
        user_id=current_user.id,
        status=status,
        search=search,
        page=page,
        per_page=per_page
    )


@router.get("/projects/{project_id}")
async def get_project(
    project_id: int,
    current_user = Depends(get_current_user),
    service: ProjectService = Depends()
):
    """Holt Projektdetails (nur wenn zugewiesen)"""
    project = await service.get_project_for_user(project_id, current_user.id)
    if not project:
        raise HTTPException(404, "Projekt nicht gefunden oder kein Zugriff")
    return project


@router.post("/time-tracking/sync")
async def sync_time_tracking(
    data: TimeTrackingSync,
    current_user = Depends(get_current_user),
    service: TimeTrackingService = Depends()
):
    """Synchronisiert automatische Zeiterfassung"""
    return await service.sync_session(
        user_id=current_user.id,
        session=data.session
    )


@router.get("/time-tracking/today")
async def get_today_time(
    current_user = Depends(get_current_user),
    service: TimeTrackingService = Depends()
):
    """Holt heute erfasste Zeit"""
    return await service.get_today_summary(current_user.id)


@router.post("/ai/chat")
async def ai_chat(
    data: AIChatRequest,
    current_user = Depends(get_current_user),
    service: AIService = Depends()
):
    """KI-Chat mit Tool-Nutzung"""
    return await service.chat(
        user_id=current_user.id,
        message=data.message,
        context=data.context,
        history=data.history
    )


@router.get("/search")
async def global_search(
    q: str,
    current_user = Depends(get_current_user),
    service: ProjectService = Depends()
):
    """Globale Suche (nur zugängliche Daten)"""
    return await service.search_user_data(
        user_id=current_user.id,
        query=q,
        limit=20
    )
```

## Zusammenfassung

Die Mitarbeiter-Oberfläche bietet:

1. **0% Bürokratie**: Automatische Zeiterfassung ohne manuelle Eingabe
2. **Fokussierte Ansicht**: Nur eigene Projekte und Kunden
3. **Geführte Workflows**: Schritt-für-Schritt-Anleitungen
4. **KI-Integration**: Kontextbezogene Hilfe und Berechnungen
5. **Effiziente Navigation**: Globale Suche, Schnellaktionen
6. **Echtzeit-Sync**: Multi-Window und Offline-Support

**Wichtige Einschränkungen:**
- Kein Zugriff auf nicht zugewiesene Projekte/Kunden
- Keine Organisations- oder Teamverwaltung
- Keine Finanz- oder Abrechnungsdaten
- Nur eigene Zeiteinträge sichtbar
