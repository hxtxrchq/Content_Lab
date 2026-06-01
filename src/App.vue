<template>
  <div class="app-container">
    
    <!-- Top System Switch Bar: Role Switcher & Branding Personalization Toggle -->
    <div class="system-switch-bar">
      <div class="logo-area">
        <span class="logo-icon">✨</span>
        <span class="logo-text">ContentLab</span>
        <span class="logo-badge">WORKSPACE EDITORIAL</span>
      </div>
      
      <!-- Current Active Role Context Switcher -->
      <div class="controls-area">
        <div class="role-selector">
          <span class="selector-icon">👁️</span>
          <select v-model="currentViewMode" class="select-input">
            <option value="agency">🏢 MODO AGENCIA</option>
            <option value="client">👤 MODO CLIENTE (Simple)</option>
          </select>
        </div>

        <div v-if="currentViewMode === 'agency'" class="agency-role-selector">
          <span class="selector-icon">⚙️</span>
          <select v-model="currentAgencyRole" class="select-input select-role">
            <option value="pm">Director / PM</option>
            <option value="cm">Content Manager</option>
            <option value="designer">Diseñador</option>
            <option value="video">Audiovisual</option>
          </select>
        </div>

        <!-- Custom Agency Branding simulated selector -->
        <div class="agency-branding-toggle">
          <span class="selector-icon">🎨</span>
          <select v-model="selectedBrandingLogo" class="select-input">
            <option value="Standard">ContentLab Teal</option>
            <option value="AgencyAlpha">Alpha Media (Naranja)</option>
            <option value="CreativeStudio">Creative Studio (Púrpura)</option>
          </select>
        </div>
      </div>
    </div>

    <!-- Main Workspace Area -->
    <div class="workspace" :style="customThemeColors">
      
      <!-- SIDEBAR (Fine, Elegant, Less Heavy) -->
      <aside class="sidebar">
        <!-- Client Active Brand Card Container -->
        <div class="sidebar-branding">
          <div class="brand-logo-container">
            <span class="brand-bullet" :style="{ backgroundColor: activeThemeColor }"></span>
            <span class="brand-org-name">{{ selectedBrandingLogo === 'Standard' ? 'ContentLab Studio' : selectedBrandingLogo === 'AgencyAlpha' ? 'Alpha Media Group' : 'Creative Studio' }}</span>
          </div>
        </div>

        <!-- Main Navigation Menu -->
        <nav class="sidebar-menu">
          <div class="menu-group">
            <button 
              @click="activeMainMenu = 'dashboard'" 
              :class="['menu-btn', { active: activeMainMenu === 'dashboard' }]"
            >
              <span class="btn-icon">📊</span> Dashboard
            </button>
            
            <button 
              @click="activeMainMenu = 'brief'" 
              :class="['menu-btn', { active: activeMainMenu === 'brief' }]"
            >
              <span class="btn-icon">📝</span> Brief &amp; Pilares
            </button>
            
            <button 
              @click="activeMainMenu = 'database'" 
              :class="['menu-btn', { active: activeMainMenu === 'database' }]"
            >
              <span class="btn-icon">📦</span> Repositorio de Marca
            </button>
          </div>
          
          <!-- Clientes Section (With visual layout context) -->
          <div class="menu-section">
            <div class="menu-section-header">Marcas / Clientes</div>
            <div class="client-sublist">
              <button 
                v-for="c in clients" 
                :key="c.id"
                @click="selectClient(c)"
                :class="['client-menu-btn', { active: activeMainMenu === 'client' && selectedClient?.id === c.id }]"
              >
                <span class="client-avatar">{{ c.avatar }}</span>
                <span class="client-name">{{ c.name }}</span>
              </button>
            </div>
          </div>

          <!-- Global Strategy Cross-Calendar for PMs only -->
          <div v-if="currentViewMode === 'agency' && currentAgencyRole === 'pm'" class="menu-section">
            <div class="menu-section-header">Estrategia Global</div>
            <button 
              @click="activeMainMenu = 'global_calendar'" 
              :class="['menu-btn', { active: activeMainMenu === 'global_calendar' }]"
            >
              <span class="btn-icon">📅</span> Calendario Cruzado
            </button>
          </div>
          
          <div class="menu-group mt-auto">
            <button 
              @click="activeMainMenu = 'notifications'" 
              :class="['menu-btn', { active: activeMainMenu === 'notifications' }]"
            >
              <span class="btn-icon">🔔</span> Notificaciones 
              <span class="badge" v-if="unreadNotificationsCount > 0">{{ unreadNotificationsCount }}</span>
            </button>
          </div>
        </nav>

        <!-- Current User Role Display Footer -->
        <div class="sidebar-user-footer">
          <div class="user-info">
            <div class="user-avatar">👤</div>
            <div>
              <div class="user-name">Alonso Creative</div>
              <div class="user-role-label">
                {{ currentViewMode === 'client' ? 'Cliente' : 'Agencia: ' + currentAgencyRole.toUpperCase() }}
              </div>
            </div>
          </div>
        </div>
      </aside>

      <!-- CONTENT BODY (Composition Editorial) -->
      <main class="content-body">
        
        <!-- MODULE: DASHBOARD -->
        <div v-if="activeMainMenu === 'dashboard'" class="module-view">
          <div class="view-header">
            <div>
              <span class="view-eyebrow">RESUMEN CREATIVO</span>
              <h1 class="view-title font-editorial">Planificación de la Semana</h1>
              <p class="view-subtitle">Foco central en aprobaciones de contenido, storyboards y rodajes activos.</p>
            </div>
            <div class="header-actions" v-if="currentViewMode === 'agency'">
              <span class="metric-summary-pill">Procesando: {{ activePostsCount }} piezas en pipeline</span>
            </div>
          </div>

          <!-- EDITORIAL DASHBOARD COMPOSITION (One Big Focus Card + Secondary Column) -->
          <div class="dashboard-editorial-grid">
            
            <!-- Left Side: Editorial Core Focus (Content, Mockups, Approvals) -->
            <div class="editorial-main-panel">
              <div class="card premium-hero-card card-primary">
                <div class="hero-card-header">
                  <div>
                    <span class="hero-badge">ESTRUCTURA DE CONTENIDO ACTIVA</span>
                    <h2>Revisión de Mockup Instagram</h2>
                  </div>
                  <button @click="selectClient(clients[0])" class="btn btn-secondary btn-sm">Ver Pizarra Completa →</button>
                </div>
                
                <div class="hero-preview-row">
                  <!-- Live Instagram Mockup Mini Frame -->
                  <div class="mini-ig-mockup">
                    <div class="mini-ig-header">
                      <span>🍗 wings_factory</span>
                      <span class="status-indicator-dot" style="background-color: var(--status-ready)"></span>
                    </div>
                    <div class="mini-ig-body">
                      <span class="mini-slide-badge">Slide 1/4</span>
                      <div class="mini-slide-content">
                        <strong>¿Eres fanático del picante real?</strong>
                        <p class="desc-text">[Mockup: Alitas crujientes con humo animado]</p>
                      </div>
                    </div>
                    <div class="mini-ig-footer">
                      <span>💬 2 Observaciones activas</span>
                    </div>
                  </div>

                  <!-- Quick Stats & Actions list -->
                  <div class="hero-quick-details">
                    <div class="quick-post-item">
                      <span class="post-type-lbl text-carousel">CARRUSEL</span>
                      <h4>🔥 Las 3 salsas más picantes</h4>
                      <p>Objetivo: Enganchar con la comunidad amante del picante.</p>
                      <div class="comment-summary-bubble">
                        <strong>Comentario Supervisor:</strong> "Me gusta el hook, que la foto se vea jugosa."
                      </div>
                    </div>

                    <div class="quick-post-item mt-3">
                      <span class="post-type-lbl text-reel">REEL</span>
                      <h4>🎥 Sonido del crujido real (ASMR)</h4>
                      <p>Escena 1: Alita saliendo de freidora con sonido crujiente.</p>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Composition list of active processes (No generic grids) -->
              <div class="editorial-list-block card">
                <h3 class="block-title">Aprobaciones y Flujo Creativo Reciente</h3>
                <div class="process-checklist-items">
                  <div class="process-item-row">
                    <span class="process-status badge badge-ready">Listo</span>
                    <div class="process-info">
                      <strong>Wings Factory - Carrusel "3 salsas más picantes"</strong>
                      <p>Esperando aprobación del Cliente | Copy &amp; Estructura listos</p>
                    </div>
                    <span class="process-time">Hace 10m</span>
                  </div>
                  <div class="process-item-row">
                    <span class="process-status badge badge-process">En proceso</span>
                    <div class="process-info">
                      <strong>Wings Factory - Reel "Sonido del crujido"</strong>
                      <p>Diseño y audiovisual agregando referencias visuales de Pinterest</p>
                    </div>
                    <span class="process-time">Hace 1h</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Right Side: Secondary Column (Agenda, KPIs and quick tools) -->
            <div class="editorial-side-panel">
              <div class="card simple-kpi-card mb-4" v-if="currentViewMode === 'agency'">
                <h3>Composición Global</h3>
                <div class="kpi-micro-row">
                  <div class="kpi-micro-item">
                    <span class="lbl">Activas</span>
                    <strong class="val">{{ clients.length }}</strong>
                  </div>
                  <div class="kpi-micro-item">
                    <span class="lbl">Listo</span>
                    <strong class="val">{{ countPostsByStatus('Listo') }}</strong>
                  </div>
                  <div class="kpi-micro-item">
                    <span class="lbl">Aprobado</span>
                    <strong class="val">{{ countPostsByStatus('Aprobado') }}</strong>
                  </div>
                </div>
              </div>

              <!-- Photoshoot Sessions Planer Preview -->
              <div class="card sessions-preview-card">
                <div class="sessions-card-header">
                  <h3>📸 Agenda de Sesión</h3>
                  <span class="tag-status">Mayo 2026</span>
                </div>
                <p class="desc">Preparativos requeridos en tienda física:</p>
                <ul class="mini-checklist">
                  <li v-for="item in storeChecklist" :key="item.id">
                    <input type="checkbox" v-model="item.completed" />
                    <span>{{ item.label.substring(0, 45) }}...</span>
                  </li>
                </ul>
                <button @click="activeClientTab = 'sesiones'; selectClient(clients[0])" class="btn btn-secondary btn-xs mt-3 w-100">Ver Checklist Detallado →</button>
              </div>

              <!-- Direct link to active brief pillars -->
              <div class="card brief-action-promo mt-4">
                <h4>Generador de Briefing</h4>
                <p>Crea pilares de contenido y distribuye el mix de publicaciones con IA.</p>
                <button @click="activeMainMenu = 'brief'" class="btn btn-primary btn-sm mt-3 w-100">Crear Estrategia con IA</button>
              </div>
            </div>

          </div>
        </div>

        <!-- MODULE: BRIEF & PILARES -->
        <div v-if="activeMainMenu === 'brief'" class="module-view">
          <div class="view-header">
            <div>
              <span class="view-eyebrow">DIRECCIÓN ESTRATÉGICA</span>
              <h1 class="view-title font-editorial">Briefing &amp; Pilares</h1>
              <p class="view-subtitle">Cálculo de mix de contenidos inteligentes y definición de pilares de marca.</p>
            </div>
          </div>

          <div class="tabs-container">
            <button @click="activeBriefTab = 'new'" :class="['tab-item', { active: activeBriefTab === 'new' }]">
              📝 Llenar Nuevo Brief
            </button>
            <button @click="activeBriefTab = 'history'" :class="['tab-item', { active: activeBriefTab === 'history' }]">
              📁 Historial de Briefs ({{ briefsHistory.length }})
            </button>
          </div>

          <!-- NEW BRIEF TAB -->
          <div v-if="activeBriefTab === 'new'" class="card brief-form-card mt-3">
            <h3 class="card-title">Completa el perfil de tu marca</h3>
            
            <div class="form-grid">
              <div class="form-group">
                <label class="form-label">Nombre del Cliente / Marca</label>
                <input type="text" v-model="newBrief.clientName" placeholder="Ej: Wings Heaven" class="form-control" />
              </div>
              <div class="form-group">
                <label class="form-label">Giro de Negocio</label>
                <select v-model="newBrief.niche" class="form-control">
                  <option value="restaurante">Restaurante / Comida Rápida</option>
                  <option value="fitness">Fitness / Salud</option>
                  <option value="saas">Tecnología / SaaS</option>
                  <option value="educacion">Educación / Consultoría</option>
                </select>
              </div>
              <div class="form-group full-width">
                <label class="form-label">Objetivo de la Campaña de Contenido</label>
                <textarea v-model="newBrief.goal" placeholder="¿Qué quieres lograr? Ej: Posicionar nuestras alitas crujientes y ganar reconocimiento local." class="form-control" rows="3"></textarea>
              </div>
              <div class="form-group">
                <label class="form-label">Publicaciones al Mes Requeridas</label>
                <input type="number" v-model="newBrief.monthlyPosts" class="form-control" />
              </div>
              <div class="form-group">
                <label class="form-label">Tono de Comunicación</label>
                <select v-model="newBrief.tone" class="form-control">
                  <option value="divertido">Divertido &amp; Hambriento</option>
                  <option value="profesional">Profesional &amp; Corporativo</option>
                  <option value="educativo">Informativo &amp; Educativo</option>
                </select>
              </div>
            </div>

            <div class="form-actions">
              <button @click="generateBriefPillars" class="btn btn-primary" :disabled="!newBrief.clientName">
                ⚡ Analizar con IA y Generar Pilares
              </button>
            </div>

            <!-- AI Output pillars display inside brief new (Visual Bars, Chips) -->
            <div v-if="generatedPillarsResult" class="ai-pillars-result">
              <h4 class="pillars-title">✨ Propuesta de Pilares de Contenido Generada con IA:</h4>
              <div class="pillars-layout">
                <div class="pillars-chart-container">
                  <div class="pillar-chart-bar" v-for="p in generatedPillarsResult.pillars" :key="p.name">
                    <div class="bar-header-row">
                      <span class="bar-name-chip" :style="{ borderLeft: '4px solid ' + p.color }">{{ p.name }}</span>
                      <strong class="bar-percentage">{{ p.percentage }}%</strong>
                    </div>
                    <div class="bar-track">
                      <div class="bar-fill" :style="{ width: p.percentage + '%', backgroundColor: p.color }"></div>
                    </div>
                    <p class="pillar-desc">{{ p.desc }}</p>
                  </div>
                </div>
                
                <div class="pillars-summary-card">
                  <h5>Estructura Mensual</h5>
                  <p>Basado en el brief de <strong>{{ generatedPillarsResult.clientName }}</strong> con meta de <strong>{{ generatedPillarsResult.monthlyPosts }} posts mensuales</strong>:</p>
                  <ul class="monthly-list">
                    <li v-for="p in generatedPillarsResult.pillars" :key="p.name">
                      <span class="bullet" :style="{ backgroundColor: p.color }"></span>
                      <span>{{ Math.round((p.percentage/100) * generatedPillarsResult.monthlyPosts) }} publicaciones de {{ p.name }}</span>
                    </li>
                  </ul>
                  <button @click="saveBriefToHistory" class="btn btn-secondary btn-sm mt-3 w-100">
                    📁 Guardar y Crear Carpetas Independientes
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- BRIEF HISTORY TAB -->
          <div v-if="activeBriefTab === 'history'" class="brief-history-list mt-3">
            <div v-for="b in briefsHistory" :key="b.id" class="card history-brief-item">
              <div class="history-item-header">
                <div>
                  <h4>📝 Briefing: {{ b.clientName }}</h4>
                  <span class="badge badge-idea">{{ b.niche.toUpperCase() }}</span>
                </div>
                <div class="history-date">{{ b.date }}</div>
              </div>
              <p class="history-desc"><strong>Objetivo:</strong> {{ b.goal }}</p>
              
              <div class="history-pillars-list">
                <div v-for="pil in b.pillars" :key="pil.name" class="pillar-tag">
                  <span class="tag-color-bullet" :style="{ backgroundColor: pil.color }"></span>
                  {{ pil.name }}: <strong>{{ pil.percentage }}%</strong>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- MODULE: REPOSITORIO DE MARCA -->
        <div v-if="activeMainMenu === 'database'" class="module-view">
          <div class="view-header">
            <div>
              <span class="view-eyebrow">REPOSITORIO CENTRAL</span>
              <h1 class="view-title font-editorial">Repositorio de Marca</h1>
              <p class="view-subtitle">Guías estratégicas indispensables de cada cliente.</p>
            </div>
          </div>
          <div class="grid-repo">
            <div class="card repo-card" v-for="c in clients" :key="c.id">
              <div class="repo-card-head">
                <span class="client-avatar">{{ c.avatar }}</span>
                <h4>{{ c.name }}</h4>
              </div>
              <div class="repo-body-links">
                <div class="repo-item-link">📁 Manual de Identidad Corporativa.pdf</div>
                <div class="repo-item-link">🎨 Paleta Hexadecimal: #D32F2F, #FFC107, #212121</div>
                <div class="repo-item-link">🗣️ Guía de Tono y Arquetipo de Marca</div>
                <div class="repo-item-link">👥 Buyer Persona: Universitario hambriento, fan de promos</div>
              </div>
            </div>
          </div>
        </div>

        <!-- MODULE: NOTIFICATIONS -->
        <div v-if="activeMainMenu === 'notifications'" class="module-view">
          <div class="view-header">
            <div>
              <span class="view-eyebrow">AVISOS Y ALERTAS</span>
              <h1 class="view-title font-editorial">Notificaciones</h1>
              <p class="view-subtitle">Control de revisiones, comentarios y tareas de producción.</p>
            </div>
            <button @click="markAllNotificationsAsRead" class="btn btn-secondary btn-sm">Marcar todas como leídas</button>
          </div>

          <div class="notifications-layout">
            <div class="notifications-column">
              <h3 class="col-title">🏢 Comunicación Interna (Agencia)</h3>
              <div class="notifications-list">
                <div 
                  v-for="n in notifications.filter(x => x.type === 'agency')" 
                  :key="n.id" 
                  :class="['notif-item', { unread: !n.read }]"
                  @click="n.read = true"
                >
                  <div class="notif-header">
                    <span class="notif-bullet"></span>
                    <span class="notif-time">{{ n.time }}</span>
                  </div>
                  <p class="notif-msg">{{ n.msg }}</p>
                </div>
              </div>
            </div>

            <div class="notifications-column">
              <h3 class="col-title">👤 Seguimiento de Clientes</h3>
              <div class="notifications-list">
                <div 
                  v-for="n in notifications.filter(x => x.type === 'client')" 
                  :key="n.id" 
                  :class="['notif-item', { unread: !n.read }]"
                  @click="n.read = true"
                >
                  <div class="notif-header">
                    <span class="notif-bullet"></span>
                    <span class="notif-time">{{ n.time }}</span>
                  </div>
                  <p class="notif-msg">{{ n.msg }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- MODULE: GLOBAL CALENDAR (PM/Director Agency Only) -->
        <div v-if="activeMainMenu === 'global_calendar' && currentViewMode === 'agency'" class="module-view">
          <div class="view-header">
            <div>
              <span class="view-eyebrow">CRONOGRAMA EDITORIAL</span>
              <h1 class="view-title font-editorial">Calendario Cruzado Multi-Marca</h1>
              <p class="view-subtitle">Control estratégico de publicaciones simultáneas de todas tus marcas.</p>
            </div>
          </div>

          <div class="card">
            <div class="cross-calendar-grid">
              <div class="cross-cal-day" v-for="day in weekDays" :key="day">
                <div class="cross-day-header">{{ day }}</div>
                <div class="cross-day-posts-container">
                  <div 
                    v-for="post in getPostsForDay(day)" 
                    :key="post.id"
                    class="cross-post-pill"
                    :style="{ borderLeftColor: getStatusColor(post.status) }"
                  >
                    <span class="cross-post-brand">{{ post.clientName }}</span>
                    <span class="cross-post-type">{{ post.type }}</span>
                    <span class="cross-post-title">{{ post.title }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- MODULE: CLIENT SPECIFIC VIEW -->
        <div v-if="activeMainMenu === 'client' && selectedClient" class="module-view">
          <div class="client-detail-header">
            <div class="client-logo-title">
              <span class="client-detail-avatar">{{ selectedClient.avatar }}</span>
              <div>
                <h1 class="client-detail-name font-editorial">{{ selectedClient.name }}</h1>
                <p class="client-detail-niche">Rubro: {{ selectedClient.niche }}</p>
              </div>
            </div>

            <!-- Contextual Header Actions -->
            <div class="client-header-actions">
              <button @click="downloadClientPDF" class="btn btn-secondary">
                📥 Descargar Estructuras PDF
              </button>
            </div>
          </div>

          <!-- Client Submenu Tabs (Integrated UI kit tabs) -->
          <div class="tabs-container mb-3">
            <button 
              v-for="tab in clientTabsFiltered" 
              :key="tab.id"
              @click="activeClientTab = tab.id"
              :class="['tab-item', { active: activeClientTab === tab.id }]"
            >
              {{ tab.label }}
            </button>
          </div>

          <!-- SUBTAB: RESUMEN -->
          <div v-if="activeClientTab === 'resumen'" class="subtab-view mt-3">
            <div class="grid-resumen">
              <div class="card">
                <h3 class="card-title">Resumen de Contrato</h3>
                <div class="resumen-metrics">
                  <div class="metric-row">
                    <span class="metric-label">Publicaciones al mes pactadas:</span>
                    <strong class="metric-value">12 Posts</strong>
                  </div>
                  <div class="metric-row">
                    <span class="metric-label">Pilares de Contenido:</span>
                    <div class="pilars-badges">
                      <span class="badge p-prod">Producto: 60%</span>
                      <span class="badge p-life">Lifestyle: 10%</span>
                      <span class="badge p-edu">Educativo: 20%</span>
                      <span class="badge p-prom">Promos: 10%</span>
                    </div>
                  </div>
                </div>
              </div>

              <div class="card feed-evolution-card">
                <div class="card-header">
                  <h3 class="card-title">📈 Evolución Visual de Marca</h3>
                  <p class="card-desc">Historial inicial vs evolución planificada actual en Instagram.</p>
                </div>
                <div class="feed-split-view">
                  <div class="feed-column">
                    <h4 class="feed-col-title title-before">Feed Original (Antes)</h4>
                    <div class="mockup-feed-grid">
                      <div class="feed-item bg-gray-dark"><span class="lbl-feed">Diseño plano alitas</span><span class="stats-mini">❤️ 15 💬 2</span></div>
                      <div class="feed-item bg-gray-dark"><span class="lbl-feed">Post de solo texto</span><span class="stats-mini">❤️ 5 💬 0</span></div>
                    </div>
                  </div>
                  <div class="feed-column">
                    <h4 class="feed-col-title title-after">Feed Diseñado (Ahora)</h4>
                    <div class="mockup-feed-grid grid-after">
                      <div class="feed-item bg-brand-colored"><span class="lbl-feed">Carrusel: Especial de Alitas</span><span class="stats-mini">❤️ 212 💬 40</span></div>
                      <div class="feed-item bg-brand-colored"><span class="lbl-feed">Reel: Crujientes Momentos</span><span class="stats-mini">❤️ 503 💬 92</span></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- SUBTAB: CONTENIDO (CORE PIZARRA ESTRUCTURADA) -->
          <div v-if="activeClientTab === 'contenido'" class="subtab-view mt-3">
            <div class="contenido-grid">
              
              <!-- Left Column: Content list selector (Structured Posts) -->
              <div class="contenido-sidebar card">
                <h4 class="sidebar-title">Estructuras de Contenido</h4>
                <div class="posts-list">
                  <button 
                    v-for="post in selectedClient.posts" 
                    :key="post.id"
                    @click="activePostId = post.id; activeCarouselSlide = 0"
                    :class="['post-list-btn', { active: activePostId === post.id }]"
                  >
                    <div class="post-btn-meta">
                      <span class="post-type-tag">{{ post.type }}</span>
                      <span class="post-status-bullet" :style="{ backgroundColor: getStatusColor(post.status) }"></span>
                    </div>
                    <div class="post-btn-title">{{ post.title }}</div>
                    <div class="post-btn-date">{{ post.date }}</div>
                  </button>
                </div>
              </div>

              <!-- Right Column: Interactive Editor Mockup Grid -->
              <div class="contenido-viewer" v-if="currentPost">
                <div class="viewer-header card">
                  <div class="post-main-details">
                    <span class="badge" :style="{ backgroundColor: getStatusColor(currentPost.status), color: '#fff' }">
                      {{ currentPost.status.toUpperCase() }}
                    </span>
                    <h2>{{ currentPost.title }}</h2>
                    <p class="post-meta-text">📅 Publicación: <strong>{{ currentPost.date }}</strong> | 🎯 Objetivo: {{ currentPost.objective }}</p>
                  </div>

                  <div class="post-action-buttons">
                    <button @click="changePostStatus(currentPost, 'Aprobado')" class="btn btn-success btn-sm">Aprobar ✅</button>
                    <button @click="changePostStatus(currentPost, 'En proceso')" class="btn btn-warning btn-sm">Solicitar Cambios ✏️</button>
                  </div>
                </div>

                <!-- CAROUSEL MOCKUP TYPE -->
                <div v-if="currentPost.type === 'Carrusel'" class="carousel-visual-editor">
                  <div class="carousel-preview-column">
                    <h3 class="viewer-section-title">Visualización de Slide</h3>
                    
                    <!-- Instagram Slide Frame Mockup -->
                    <div class="instagram-frame">
                      <div class="ig-header">
                        <span class="ig-avatar">🍗</span>
                        <div class="ig-user-info">
                          <span class="ig-username">{{ selectedClient.name.toLowerCase().replace(' ', '_') }}</span>
                          <span class="ig-location">Patrocinado</span>
                        </div>
                        <span class="ig-dots">•••</span>
                      </div>

                      <!-- Slide Content body -->
                      <div class="ig-slide-body">
                        <div class="slide-indicator">{{ activeCarouselSlide + 1 }} / {{ currentPost.slides.length }}</div>
                        
                        <div class="slide-designer-canvas" :style="{ backgroundColor: activeCarouselSlideContent.bgColor || '#f3f4f6' }">
                          <h4 class="slide-canvas-hook">{{ activeCarouselSlideContent.hook }}</h4>
                          <p class="slide-canvas-concept">{{ activeCarouselSlideContent.concept }}</p>
                          <div class="slide-bottom-guide">[Estructura Conceptual - No diseño final]</div>
                        </div>
                      </div>

                      <!-- Instagram interactive action icons -->
                      <div class="ig-footer-actions">
                        <div class="ig-action-icons">
                          <span>❤️</span>
                          <span>💬</span>
                          <span>✈️</span>
                        </div>
                        <div class="ig-carousel-dots">
                          <span 
                            v-for="(slide, idx) in currentPost.slides" 
                            :key="idx" 
                            :class="['ig-dot', { active: idx === activeCarouselSlide }]"
                          ></span>
                        </div>
                      </div>
                    </div>

                    <!-- Slide Navigator Buttons -->
                    <div class="slide-navigator">
                      <button @click="prevSlide" :disabled="activeCarouselSlide === 0" class="btn btn-secondary btn-sm">◀ Anterior</button>
                      <span class="slide-indicator-label">Slide {{ activeCarouselSlide + 1 }}</span>
                      <button @click="nextSlide" :disabled="activeCarouselSlide === currentPost.slides.length - 1" class="btn btn-secondary btn-sm">Siguiente ▶</button>
                    </div>
                  </div>

                  <!-- Comments and Feedback side panel for current Carousel Slide -->
                  <div class="carousel-feedback-column card">
                    <h3 class="feedback-title">💬 Observaciones en Slide {{ activeCarouselSlide + 1 }}</h3>
                    <p class="feedback-desc">Comentarios del supervisor de cuentas para esta slide.</p>
                    
                    <div class="comments-scroller">
                      <div 
                        v-for="comment in activeCarouselSlideContent.comments" 
                        :key="comment.id" 
                        class="comment-bubble"
                      >
                        <div class="comment-author">
                          <strong>{{ comment.author }}</strong> 
                          <span class="comment-role">({{ comment.role }})</span>
                        </div>
                        <p class="comment-text">{{ comment.text }}</p>
                      </div>
                    </div>

                    <!-- Post comments input form -->
                    <div class="comment-input-area">
                      <input 
                        type="text" 
                        v-model="newCommentText" 
                        placeholder="Escribe tu observación para este slide..." 
                        class="form-control form-control-sm"
                        @keyup.enter="addCommentToActiveSlide"
                      />
                      <button @click="addCommentToActiveSlide" class="btn btn-primary btn-sm mt-2">Enviar</button>
                    </div>
                  </div>
                </div>

                <!-- REELS STORYBOARD TYPE -->
                <div v-if="currentPost.type === 'Reel'" class="reel-storyboard-editor">
                  <h3 class="viewer-section-title">🎬 Guion y Storyboard de Reel</h3>
                  
                  <div class="reel-details-grid">
                    <div class="card info-card">
                      <h5>Escena y Actuación</h5>
                      <p><strong>Lo Actuado:</strong> {{ currentPost.reelDetails.acting }}</p>
                    </div>
                    <div class="card info-card">
                      <h5>Audio y Voz en Off</h5>
                      <p><strong>Voz en off / Sonido:</strong> {{ currentPost.reelDetails.voiceover }}</p>
                    </div>
                    <div class="card info-card">
                      <h5>📌 Referencia Visual</h5>
                      <div class="pinterest-ref-link">
                        <span class="pin-icon">📌</span>
                        <a href="#" class="ref-link" @click.prevent="alert('Simulación: Abriendo referencia de Pinterest')">
                          {{ currentPost.reelDetails.pinterestRef }}
                        </a>
                      </div>
                    </div>
                  </div>

                  <div class="storyboard-scenes-table card mt-4">
                    <h4 class="table-title">🎬 Storyboard por Escena</h4>
                    <table class="simple-table">
                      <thead>
                        <tr>
                          <th>Escena</th>
                          <th>Visual / Acción</th>
                          <th>Audio / Sonido</th>
                          <th>Ref Pinterest</th>
                          <th>Estado</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="scene in currentPost.reelDetails.scenes" :key="scene.number">
                          <td><strong>Escena {{ scene.number }}</strong></td>
                          <td>{{ scene.action }}</td>
                          <td>{{ scene.audio }}</td>
                          <td><span class="badge badge-idea">Ref {{ scene.number }}</span></td>
                          <td>
                            <span class="badge badge-approved">Listado</span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>

                <!-- STORIES GRID TYPE -->
                <div v-if="currentPost.type === 'Historia'" class="stories-grid-editor">
                  <h3 class="viewer-section-title">📱 Grilla de Planificación de Historias</h3>
                  <div class="stories-grid">
                    <div v-for="story in currentPost.storiesDetails" :key="story.id" class="story-preview-card card">
                      <div class="story-preview-bg">
                        <span class="story-category-badge">{{ story.category }}</span>
                        <p class="story-hook">{{ story.hook }}</p>
                      </div>
                      <div class="story-card-meta">
                        <p class="story-desc">{{ story.concept }}</p>
                        <span class="badge badge-idea">Interactiva</span>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Creative Asset Uploader (Designer / AudioVisual tools) -->
                <div v-if="currentViewMode === 'agency'" class="card uploader-card mt-4">
                  <h4 class="card-title">📤 Panel de Carga de Recursos Finales (Exclusivo Agencia)</h4>
                  <p class="card-desc">Sube los diseños finales o videos terminados para la revisión definitiva del cliente.</p>
                  
                  <div class="upload-zone">
                    <div class="upload-placeholder">
                      <span>📁 Arrastra y suelta el archivo aquí o haz clic para buscar</span>
                      <small class="upload-help">Formatos aceptados: PNG, JPG, MP4. Máximo 50MB.</small>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- SUBTAB: CALENDARIO -->
          <div v-if="activeClientTab === 'calendario'" class="subtab-view mt-3">
            <div class="card">
              <div class="calendar-controls">
                <h3>Calendario de Contenidos - {{ selectedClient.name }}</h3>
                <div class="calendar-legend">
                  <span class="leg-item"><span class="bullet" style="background-color: var(--status-idea)"></span> Idea</span>
                  <span class="leg-item"><span class="bullet" style="background-color: var(--status-process)"></span> En proceso</span>
                  <span class="leg-item"><span class="bullet" style="background-color: var(--status-ready)"></span> Listo</span>
                  <span class="leg-item"><span class="bullet" style="background-color: var(--status-approved)"></span> Aprobado</span>
                </div>
              </div>

              <!-- Content calendar grid weekly representation -->
              <div class="client-calendar-grid">
                <div v-for="post in selectedClient.posts" :key="post.id" class="calendar-cell card">
                  <div class="cell-header">
                    <span class="cell-date">📅 {{ post.date }}</span>
                    <span class="cell-status-bullet" :style="{ backgroundColor: getStatusColor(post.status) }"></span>
                  </div>
                  <div class="cell-body">
                    <span class="cell-type">{{ post.type }}</span>
                    <h5 class="cell-title">{{ post.title }}</h5>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- SUBTAB: SESIONES DE FOTO Y VIDEO -->
          <div v-if="activeClientTab === 'sesiones'" class="subtab-view mt-3">
            <div class="sesiones-layout">
              
              <!-- Photos Grouping -->
              <div class="card sesion-col">
                <h3 class="col-title-primary">📸 Sesión de Fotos Planificadas</h3>
                <p class="col-subtitle">Distribución organizada de tomas de producto y branding.</p>
                
                <div class="session-block">
                  <div class="session-category-header">Fotos de Producto - Alitas Crujientes</div>
                  <div class="session-card-details">
                    <p><strong>Ropa/Vestimenta Ref:</strong> Casual, colores de marca (delantal rojo/amarillo).</p>
                    <p><strong>Persona Ideal / Modelo:</strong> Estudiante joven disfrutando de comida.</p>
                    <p><strong>Fotos a realizar:</strong> Foto 1 (Primer plano alitas bañadas en salsa), Foto 2 (Caja de alitas con logo en primer plano).</p>
                  </div>
                </div>
              </div>

              <!-- Videos Grouping -->
              <div class="card sesion-col">
                <h3 class="col-title-primary">🎥 Sesión de Videos Planificadas</h3>
                <p class="col-subtitle">Organización de tomas de reel y storyboards dinámicos.</p>
                
                <div class="session-block">
                  <div class="session-category-header">Video 1 - El crujido de la primera mordida</div>
                  <div class="session-card-details">
                    <p><strong>Ropa/Vestimenta Ref:</strong> Camisa de cuadros casual, colores neutros.</p>
                    <p><strong>Persona Ideal:</strong> Persona expresiva, fanático del picante.</p>
                    <p><strong>Tomas clave:</strong> Primer plano comiendo y mostrando el sonido crujiente.</p>
                  </div>
                </div>
              </div>

              <!-- Client Checklist shop panel (What they must prepare in the store) -->
              <div class="card checklist-shop-card">
                <h3 class="col-title-primary">🏪 Responsabilidad de Tienda (Listado de Preparativos)</h3>
                <p class="col-subtitle">Lo que el cliente debe tener listo antes del rodaje.</p>
                
                <div class="checklist-items-container">
                  <div 
                    v-for="item in storeChecklist" 
                    :key="item.id" 
                    :class="['checklist-item-row', { checked: item.completed }]"
                  >
                    <input 
                      type="checkbox" 
                      v-model="item.completed" 
                      class="check-control"
                    />
                    <span class="checklist-label-text">{{ item.label }}</span>
                  </div>
                </div>
              </div>

            </div>
          </div>

          <!-- SUBTAB: NUBE -->
          <div v-if="activeClientTab === 'nube'" class="subtab-view mt-3">
            <div class="card cloud-storage-card">
              <div class="cloud-header">
                <h3>📁 Almacenamiento Nube</h3>
                <p class="cloud-desc">Organización interna de archivos por cliente y mes.</p>
              </div>

              <div class="cloud-folder-tree">
                <div class="folder-row">
                  <span class="folder-icon">📂</span>
                  <strong>Mayo 2026</strong>
                </div>
                <div class="folder-contents">
                  <div class="file-row">
                    <span class="file-icon">🖼️</span>
                    <span>foto_alitas_picantes_ref.jpg</span>
                    <span class="file-size">1.2 MB</span>
                  </div>
                  <div class="file-row">
                    <span class="file-icon">🎥</span>
                    <span>reel_sonido_crujiente_ref.mp4</span>
                    <span class="file-size">14.8 MB</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- SUBTAB: PROGRAMACIÓN -->
          <div v-if="activeClientTab === 'programacion'" class="subtab-view mt-3">
            <div class="card text-center">
              <h3>🔗 Conexión &amp; Programación de Redes</h3>
              <p>Simulador de calendarización automática de posts en plataformas conectadas (Instagram, Facebook, TikTok).</p>
              
              <div class="mt-4">
                <span class="badge badge-approved">Instagram Conectado</span>
                <span class="badge badge-idea ml-2">TikTok (Pendiente)</span>
              </div>
            </div>
          </div>

        </div>

      </main>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      currentViewMode: 'agency', // agency or client
      currentAgencyRole: 'pm', // pm, cm, designer, video
      selectedBrandingLogo: 'Standard',
      activeMainMenu: 'dashboard', // dashboard, brief, database, client, notifications
      activeClientTab: 'contenido', // resumen, contenido, calendario, sesiones, nube, programacion
      activeBriefTab: 'new', // new, history
      
      // Selected brand context
      selectedClient: null,
      activePostId: null,
      activeCarouselSlide: 0,
      
      // Post comment input
      newCommentText: '',

      // New Brief Form State
      newBrief: {
        clientName: '',
        niche: 'restaurante',
        goal: '',
        monthlyPosts: 12,
        tone: 'divertido'
      },
      
      generatedPillarsResult: null,

      // Initial mock database setup
      briefsHistory: [
        {
          id: 'brief-1',
          clientName: 'Wings Factory',
          niche: 'restaurante',
          goal: 'Establecer la calidad premium y el sabor crujiente de las alitas artesanales.',
          monthlyPosts: 12,
          tone: 'divertido',
          date: '15/05/2026',
          pillars: [
            { name: 'Producto / Foodporn', percentage: 60, color: '#0d9488', desc: 'Tomas cercanas de alitas recién cocinadas bañadas en salsas.' },
            { name: 'Lifestyle / Momentos', percentage: 10, color: '#2563eb', desc: 'Amigos compartiendo bandejas de alitas en locales.' },
            { name: 'Educativo / Datos', percentage: 20, color: '#d97706', desc: 'Origen de los chiles y tipos de salsas.' },
            { name: 'Promos / Venta Directa', percentage: 10, color: '#7c3aed', desc: 'Combos de miércoles de alitas y promociones.' }
          ]
        }
      ],

      clients: [
        {
          id: 'client-1',
          name: 'Wings Factory',
          avatar: '🍗',
          niche: 'Restaurante de Alitas',
          posts: [
            {
              id: 'post-1',
              type: 'Carrusel',
              title: '🔥 Las 3 salsas más picantes que vas a probar',
              date: 'Miércoles 04 Mayo',
              objective: 'Enganchar con la audiencia amante del picante',
              status: 'Listo',
              slides: [
                {
                  number: 1,
                  hook: '🔥 ¿Eres fanático del picante real?',
                  concept: 'Mockup visual con imagen gigante de alitas ardientes de fondo con humo animado.',
                  bgColor: '#fff8f6',
                  comments: [
                    { id: 1, author: 'Carlos (Supervisor PM)', role: 'PM', text: 'Me gusta el hook, asegúrate que las alitas se vean súper jugosas en el mockup final.' },
                    { id: 2, author: 'Sofía (Content)', role: 'CM', text: 'Listo, agregaremos el logo de Wings Factory en la esquina superior.' }
                  ]
                },
                {
                  number: 2,
                  hook: '🥵 Puesto 3: Mango Habanero',
                  concept: 'Detalle de la salsa agridulce con trozos de chile habanero fresco.',
                  bgColor: '#fffbf0',
                  comments: []
                },
                {
                  number: 3,
                  hook: '💀 Puesto 1: Apocalypse Pepper',
                  concept: 'Imagen dramática del chile fantasma. Advertencia de consumo responsable.',
                  bgColor: '#fff8f6',
                  comments: []
                },
                {
                  number: 4,
                  hook: '👉 Comenta cuál es tu nivel máximo de picante',
                  concept: 'Llamado a la acción (CTA) con cajita de comentarios animada y botón de guardar.',
                  bgColor: '#f0fdf4',
                  comments: []
                }
              ]
            },
            {
              id: 'post-2',
              type: 'Reel',
              title: '🎥 Sonido del crujido real',
              date: 'Jueves 05 Mayo',
              objective: 'Branding sensorial y antojo inmediato',
              status: 'En proceso',
              reelDetails: {
                acting: 'Persona muerde una alita crujiente cerca de la cámara. Su expresión es de satisfacción absoluta.',
                voiceover: 'Sonido crujiente hiperrealista (ASMR) seguido de música moderna con bajo fuerte.',
                pinterestRef: 'https://pinterest.com/pin/mock-alitas-asmr',
                scenes: [
                  { number: 1, action: 'Primer plano alita saliendo de la freidora con burbujas de aceite.', audio: 'Música en volumen bajo, sonido de aceite hirviendo.', ref: 'ref-1' },
                  { number: 2, action: 'Mordisco en cámara superlenta con zoom al crujido.', audio: 'ASMR Crujido nítido amplificado al 200%.', ref: 'ref-2' }
                ]
              }
            },
            {
              id: 'post-3',
              type: 'Historia',
              title: '📱 Encuesta Interactiva de Picantes',
              date: 'Viernes 06 Mayo',
              objective: 'Aumentar el engagement interactivo del viernes',
              status: 'Idea',
              storiesDetails: [
                { id: 'st-1', category: 'Interacción', hook: '¿Te atreverías a probar nuestra salsa Apocalipsis? 🌶️', concept: 'Sticker de encuesta con opciones Sí / No' }
              ]
            }
          ]
        }
      ],

      storeChecklist: [
        { id: 1, label: 'Tener preparadas 10 alitas crujientes recién bañadas para la toma de producto a las 4 PM', completed: false },
        { id: 2, label: 'Limpiar mesa rústica de madera cerca del ventanal para luz natural', completed: false },
        { id: 3, label: 'Contar con envases corporativos limpios y listos', completed: false }
      ],

      weekDays: ['Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado', 'Domingo'],

      notifications: [
        { id: 1, type: 'agency', msg: 'Diseñador subió mockup final para post "Las 3 salsas más picantes".', time: 'Hace 10 min', read: false },
        { id: 2, type: 'client', msg: 'Cliente solicitó cambios en slide 1 del post de Alitas.', time: 'Hace 1 hora', read: false },
        { id: 3, type: 'agency', msg: 'Recordatorio: Subir storyboard de Reel para aprobación antes de la sesión.', time: 'Hace 3 horas', read: true }
      ]
    }
  },
  computed: {
    customThemeColors() {
      if (this.selectedBrandingLogo === 'AgencyAlpha') {
        return { '--accent-color': '#f97316', '--accent-hover': '#ea580c' }; // Orange Accent
      }
      if (this.selectedBrandingLogo === 'CreativeStudio') {
        return { '--accent-color': '#8b5cf6', '--accent-hover': '#7c3aed' }; // Purple Accent
      }
      return { '--accent-color': '#0d9488', '--accent-hover': '#0f766e' }; // Teal Standard
    },
    activeThemeColor() {
      if (this.selectedBrandingLogo === 'AgencyAlpha') return '#f97316';
      if (this.selectedBrandingLogo === 'CreativeStudio') return '#8b5cf6';
      return '#0d9488';
    },
    clientTabsFiltered() {
      return [
        { id: 'resumen', label: '📊 Resumen' },
        { id: 'contenido', label: '👀 Estructura & Mockups' },
        { id: 'calendario', label: '📅 Calendario' },
        { id: 'sesiones', label: '📸 Sesiones & Checklist' },
        { id: 'nube', label: '📁 Nube' },
        { id: 'programacion', label: '🔗 Programación' }
      ]
    },
    currentPost() {
      if (!this.selectedClient || !this.activePostId) return null;
      return this.selectedClient.posts.find(p => p.id === this.activePostId);
    },
    activeCarouselSlideContent() {
      if (!this.currentPost || this.currentPost.type !== 'Carrusel') return {};
      return this.currentPost.slides[this.activeCarouselSlide] || {};
    },
    unreadNotificationsCount() {
      return this.notifications.filter(n => !n.read).length;
    },
    activePostsCount() {
      return this.clients.reduce((acc, c) => acc + c.posts.length, 0);
    }
  },
  methods: {
    selectClient(client) {
      this.selectedClient = client;
      this.activeMainMenu = 'client';
      if (client.posts && client.posts.length > 0) {
        this.activePostId = client.posts[0].id;
        this.activeCarouselSlide = 0;
      } else {
        this.activePostId = null;
      }
    },
    countPostsByStatus(status) {
      return this.clients.reduce((acc, c) => {
        return acc + c.posts.filter(p => p.status === status).length;
      }, 0);
    },
    getPostsForDay(day) {
      const results = [];
      this.clients.forEach(c => {
        c.posts.forEach(p => {
          if (p.date.includes(day)) {
            results.push({
              id: p.id,
              clientName: c.name,
              type: p.type,
              title: p.title,
              status: p.status
            });
          }
        });
      });
      return results;
    },
    getStatusColor(status) {
      switch (status) {
        case 'Idea': return 'var(--status-idea)';
        case 'En proceso': return 'var(--status-process)';
        case 'Listo': return 'var(--status-ready)';
        case 'Aprobado': return 'var(--status-approved)';
        case 'Publicado': return 'var(--status-published)';
        default: return '#ccc';
      }
    },
    nextSlide() {
      if (this.activeCarouselSlide < this.currentPost.slides.length - 1) {
        this.activeCarouselSlide++;
      }
    },
    prevSlide() {
      if (this.activeCarouselSlide > 0) {
        this.activeCarouselSlide--;
      }
    },
    addCommentToActiveSlide() {
      if (!this.newCommentText.trim()) return;
      const slide = this.activeCarouselSlideContent;
      if (slide && slide.comments) {
        slide.comments.push({
          id: Date.now(),
          author: this.currentViewMode === 'client' ? 'Cliente (Restaurante)' : 'Agencia (' + this.currentAgencyRole.toUpperCase() + ')',
          role: this.currentViewMode === 'client' ? 'Cliente' : this.currentAgencyRole.toUpperCase(),
          text: this.newCommentText
        });
        this.newCommentText = '';
        
        this.notifications.unshift({
          id: Date.now() + 1,
          type: this.currentViewMode === 'client' ? 'agency' : 'client',
          msg: `Nuevo comentario en slide ${this.activeCarouselSlide + 1} de "${this.currentPost.title}"`,
          time: 'Ahora mismo',
          read: false
        });
      }
    },
    changePostStatus(post, newStatus) {
      post.status = newStatus;
      this.notifications.unshift({
        id: Date.now(),
        type: 'agency',
        msg: `El post "${post.title}" cambió de estado a: ${newStatus}`,
        time: 'Hace un momento',
        read: false
      });
      alert(`Estado del post modificado a: ${newStatus}`);
    },
    generateBriefPillars() {
      this.generatedPillarsResult = {
        clientName: this.newBrief.clientName,
        monthlyPosts: this.newBrief.monthlyPosts,
        pillars: [
          { name: 'Producto / Foodporn', percentage: 60, color: 'var(--accent-color)', desc: 'Toma principal del sabor de marca.' },
          { name: 'Lifestyle / Momentos de Marca', percentage: 10, color: '#2563eb', desc: 'Convivencia en local físico.' },
          { name: 'Educativo / Consejos', percentage: 20, color: '#d97706', desc: 'Secretos culinarios y recetas.' },
          { name: 'Venta Directa / Promos', percentage: 10, color: '#7c3aed', desc: 'Ofertas y combos semanales.' }
        ]
      };
    },
    saveBriefToHistory() {
      if (!this.generatedPillarsResult) return;
      
      const newHistoryItem = {
        id: 'brief-' + Date.now(),
        clientName: this.generatedPillarsResult.clientName,
        niche: this.newBrief.niche,
        goal: this.newBrief.goal,
        monthlyPosts: this.generatedPillarsResult.monthlyPosts,
        tone: this.newBrief.tone,
        date: new Date().toLocaleDateString(),
        pillars: JSON.parse(JSON.stringify(this.generatedPillarsResult.pillars))
      };
      
      this.briefsHistory.push(newHistoryItem);
      
      this.clients.push({
        id: 'client-' + Date.now(),
        name: this.generatedPillarsResult.clientName,
        avatar: '✨',
        niche: this.newBrief.niche === 'restaurante' ? 'Restaurante / Comida' : 'Rubro Nuevo',
        posts: [
          {
            id: 'post-new-1',
            type: 'Carrusel',
            title: '🔥 Lanzamiento de Marca con IA',
            date: 'Lunes 15 Mayo',
            objective: 'Presentar pilares al público',
            status: 'Idea',
            slides: [
              { number: 1, hook: '¡Llegamos a la ciudad!', concept: 'Mockup de bienvenida con paleta del cliente.', bgColor: '#fff', comments: [] }
            ]
          }
        ]
      });

      alert('¡Brief guardado con éxito! Se ha creado una carpeta de cliente independiente.');
      this.activeBriefTab = 'history';
    },
    downloadClientPDF() {
      alert('📄 PDF Descargable Generado: Se descargó el resumen estructurado de las publicaciones del cliente.');
    },
    markAllNotificationsAsRead() {
      this.notifications.forEach(n => n.read = true);
    }
  },
  mounted() {
    if (this.clients.length > 0) {
      this.selectClient(this.clients[0]);
    }
  }
}
</script>

<style scoped>
/* Main shell layout */
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: var(--bg-main);
  overflow: hidden;
  font-family: var(--font-main);
}

/* System switches topbar */
.system-switch-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 24px;
  background-color: #ffffff;
  border-bottom: 1px solid var(--border-color);
  z-index: 100;
}

.logo-area {
  display: flex;
  align-items: center;
  gap: 8px;
}
.logo-icon {
  font-size: 1.35rem;
}
.logo-text {
  font-weight: 800;
  font-size: 1.25rem;
  letter-spacing: -0.5px;
  color: var(--text-main);
}
.logo-badge {
  font-size: 0.7rem;
  font-weight: 600;
  padding: 2px 8px;
  background-color: #f4f4f0;
  border-radius: 99px;
  color: var(--text-muted);
}

.controls-area {
  display: flex;
  gap: 20px;
  align-items: center;
}

.role-selector, .agency-role-selector, .agency-branding-toggle {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-muted);
}

.select-input {
  padding: 8px 12px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  font-family: inherit;
  font-size: 0.8rem;
  background-color: #fff;
  cursor: pointer;
  outline: none;
  font-weight: 600;
  color: var(--text-main);
  transition: all 0.2s ease;
}
.select-input:hover {
  border-color: var(--text-muted);
}
.select-input:focus {
  border-color: var(--accent-color);
  box-shadow: 0 0 0 3px rgba(15, 118, 110, 0.1);
}

.select-role {
  border-color: var(--accent-color);
  font-weight: 600;
}

/* Workspace Frame */
.workspace {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* Sidebar styling (Finer and elegant) */
.sidebar {
  width: 220px;
  background-color: var(--bg-sidebar);
  border-right: 1px solid var(--border-color);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.sidebar-branding {
  padding: 20px 24px;
}

.brand-logo-container {
  display: flex;
  align-items: center;
  gap: 8px;
}
.brand-bullet {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}
.brand-org-name {
  font-weight: 700;
  font-size: 0.85rem;
  letter-spacing: -0.2px;
}

.sidebar-menu {
  padding: 8px 12px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  flex: 1;
  overflow-y: auto;
}

.menu-group {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.menu-btn {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 10px 14px;
  background: none;
  border: 1px solid transparent;
  border-radius: var(--radius-md);
  text-align: left;
  font-size: 0.85rem;
  font-weight: 600;
  font-family: inherit;
  color: var(--text-muted);
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}
.menu-btn:hover {
  background-color: rgba(29, 29, 26, 0.03);
  color: var(--text-main);
}
.menu-btn.active {
  background-color: #ffffff;
  color: var(--text-main);
  font-weight: 700;
  box-shadow: var(--shadow-sm);
  border: 1px solid var(--border-color);
}

.menu-section {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.menu-section-header {
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  color: #a1a19a;
  letter-spacing: 0.5px;
  padding-left: 12px;
  margin-bottom: 2px;
}

.client-sublist {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.client-menu-btn {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 8px 12px;
  background: none;
  border: 1px solid transparent;
  border-radius: var(--radius-md);
  text-align: left;
  font-size: 0.8rem;
  font-weight: 600;
  font-family: inherit;
  color: var(--text-muted);
  cursor: pointer;
  transition: all 0.2s ease;
}
.client-menu-btn:hover {
  background-color: rgba(29, 29, 26, 0.03);
  color: var(--text-main);
}
.client-menu-btn.active {
  background-color: #ffffff;
  color: var(--text-main);
  font-weight: 700;
  box-shadow: var(--shadow-sm);
  border: 1px solid var(--border-color);
}

.client-avatar {
  font-size: 0.95rem;
}

.badge {
  font-size: 0.65rem;
  font-weight: 700;
  padding: 2px 6px;
  background-color: var(--accent-color);
  color: #fff;
  border-radius: 99px;
  margin-left: auto;
}

.sidebar-user-footer {
  padding: 16px 20px;
  border-top: 1px solid var(--border-color);
  background-color: rgba(29, 29, 26, 0.01);
}
.user-info {
  display: flex;
  align-items: center;
  gap: 10px;
}
.user-avatar {
  font-size: 1.2rem;
}
.user-name {
  font-weight: 700;
  font-size: 0.8rem;
}
.user-role-label {
  font-size: 0.7rem;
  color: var(--text-muted);
}

/* Content Area layout */
.content-body {
  flex: 1;
  padding: 32px;
  overflow-y: auto;
}

.module-view {
  display: flex;
  flex-direction: column;
  gap: 28px;
}

.view-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}
.view-eyebrow {
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 1px;
  text-transform: uppercase;
  color: var(--accent-color);
}
.view-title {
  font-size: 1.75rem;
  font-weight: 800;
  letter-spacing: -0.5px;
  margin-top: 2px;
}
.view-subtitle {
  font-size: 0.85rem;
  color: var(--text-muted);
  margin-top: 2px;
}
.font-editorial {
  font-family: var(--font-title);
  color: var(--text-main);
  font-weight: 700;
}

.metric-summary-pill {
  font-size: 0.75rem;
  padding: 4px 12px;
  background-color: var(--accent-light);
  color: var(--accent-color);
  border-radius: 99px;
  font-weight: 700;
}

/* EDITORIAL DASHBOARD COMPOSITION */
.dashboard-editorial-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
}

.editorial-main-panel {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.editorial-side-panel {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* Premium Focus Card */
.hero-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 20px;
}
.hero-badge {
  font-size: 0.65rem;
  font-weight: 700;
  background-color: var(--accent-light);
  color: var(--accent-color);
  padding: 3px 8px;
  border-radius: var(--radius-xs);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
.hero-card-header h2 {
  font-size: 1.25rem;
  font-weight: 800;
  margin-top: 4px;
  letter-spacing: -0.3px;
}

.hero-preview-row {
  display: grid;
  grid-template-columns: 1fr 1.2fr;
  gap: 24px;
  align-items: center;
}

/* Instagram Miniature Frame */
.mini-ig-mockup {
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  background-color: #fff;
  overflow: hidden;
  box-shadow: var(--shadow-sm);
}
.mini-ig-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  font-size: 0.75rem;
  font-weight: 700;
  border-bottom: 1px solid #f4f4f0;
}
.status-indicator-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}
.mini-ig-body {
  aspect-ratio: 1.1;
  background-color: #fff8f6;
  padding: 16px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.mini-slide-badge {
  font-size: 0.65rem;
  background-color: rgba(0, 0, 0, 0.05);
  padding: 2px 6px;
  border-radius: 4px;
  align-self: flex-end;
}
.mini-slide-content {
  text-align: center;
}
.mini-slide-content strong {
  font-size: 0.85rem;
  font-weight: 800;
  display: block;
  margin-bottom: 6px;
}
.mini-slide-content p {
  font-size: 0.7rem;
  color: var(--text-muted);
}
.mini-ig-footer {
  padding: 8px 12px;
  font-size: 0.7rem;
  color: var(--accent-color);
  background-color: var(--accent-light);
  font-weight: 600;
  border-top: 1px solid #f4f4f0;
}

/* Quick Post items */
.quick-post-item {
  padding-bottom: 12px;
  border-bottom: 1px dashed var(--border-color);
}
.post-type-lbl {
  font-size: 0.65rem;
  font-weight: 700;
  letter-spacing: 0.5px;
}
.text-carousel { color: #f59e0b; }
.text-reel { color: #8b5cf6; }

.quick-post-item h4 {
  font-size: 0.95rem;
  font-weight: 700;
  margin: 2px 0;
}
.quick-post-item p {
  font-size: 0.8rem;
  color: var(--text-muted);
}
.comment-summary-bubble {
  margin-top: 8px;
  padding: 8px 12px;
  background-color: #fafaf9;
  border-radius: var(--radius-sm);
  font-size: 0.75rem;
  border-left: 2px solid var(--accent-color);
}

/* Editorial recent process list */
.block-title {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 16px;
}
.process-checklist-items {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.process-item-row {
  display: flex;
  align-items: center;
  gap: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid #f4f4f0;
}

.process-info {
  flex: 1;
}
.process-info strong {
  font-size: 0.85rem;
  display: block;
}
.process-info p {
  font-size: 0.75rem;
  color: var(--text-muted);
}
.process-time {
  font-size: 0.7rem;
  color: var(--text-muted);
}

/* Simple KPIs representation */
.kpi-micro-row {
  display: flex;
  justify-content: space-between;
  margin-top: 12px;
}
.kpi-micro-item {
  display: flex;
  flex-direction: column;
}
.kpi-micro-item .lbl {
  font-size: 0.7rem;
  font-weight: 600;
  color: var(--text-muted);
}
.kpi-micro-item .val {
  font-size: 1.25rem;
  font-weight: 800;
}

/* Sessions card layout */
.sessions-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
.sessions-preview-card .desc {
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-bottom: 8px;
}
.mini-checklist {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.mini-checklist li {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 0.75rem;
}
.mini-checklist li input {
  margin-top: 3px;
}

/* Brief Promos card */
.brief-action-promo h4 {
  font-size: 1rem;
  font-weight: 700;
}
.brief-action-promo p {
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-top: 4px;
}

/* Form layout */
.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.full-width {
  grid-column: span 2;
}

.form-control {
  padding: 10px 14px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  font-family: inherit;
  font-size: 0.85rem;
  outline: none;
  background-color: #ffffff;
  transition: all 0.2s ease;
}
.form-control:focus {
  border-color: var(--accent-color);
  box-shadow: 0 0 0 3px rgba(15, 118, 110, 0.1);
}

.form-actions {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}

/* AI Pillars visual result section (Bars, Chips, Percentages) */
.ai-pillars-result {
  margin-top: 28px;
  padding-top: 24px;
  border-top: 1px dashed var(--border-color);
}
.pillars-title {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 16px;
}
.pillars-layout {
  display: grid;
  grid-template-columns: 2fr 1.2fr;
  gap: 28px;
}
.pillar-chart-bar {
  margin-bottom: 18px;
}
.bar-header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}
.bar-name-chip {
  font-size: 0.8rem;
  font-weight: 700;
  padding-left: 8px;
}
.bar-percentage {
  font-size: 0.85rem;
  font-weight: 800;
}
.bar-track {
  height: 6px;
  background-color: #f4f4f0;
  border-radius: 99px;
  overflow: hidden;
  margin-bottom: 4px;
}
.bar-fill {
  height: 100%;
  border-radius: 99px;
}
.pillar-desc {
  font-size: 0.75rem;
  color: var(--text-muted);
}
.pillars-summary-card {
  background-color: var(--bg-sidebar);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 16px;
}
.pillars-summary-card h5 {
  font-size: 0.85rem;
  font-weight: 700;
  margin-bottom: 8px;
}
.monthly-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 12px;
}
.monthly-list li {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.75rem;
}
.monthly-list li .bullet {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}

/* Client Detail Layout */
.client-detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 16px;
}
.client-logo-title {
  display: flex;
  align-items: center;
  gap: 12px;
}
.client-detail-avatar {
  font-size: 2rem;
}
.client-detail-name {
  font-size: 1.35rem;
  font-weight: 800;
}
.client-detail-niche {
  font-size: 0.8rem;
  color: var(--text-muted);
}

.client-tabs {
  display: flex;
  gap: 4px;
  border-bottom: 1px solid var(--border-color);
}
.client-tab-btn {
  padding: 8px 16px;
  background: none;
  border: none;
  border-bottom: 2px solid transparent;
  font-family: inherit;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-muted);
  cursor: pointer;
  transition: all 0.2s;
}
.client-tab-btn.active {
  border-bottom-color: var(--accent-color);
  color: var(--accent-color);
}

/* Post list sidebar & viewer grid */
.contenido-grid {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 24px;
}
.contenido-sidebar {
  padding: 16px;
}
.sidebar-title {
  font-size: 0.8rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: #a1a19a;
  margin-bottom: 12px;
}
.posts-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.post-list-btn {
  display: flex;
  flex-direction: column;
  width: 100%;
  padding: 12px;
  background-color: #fafaf9;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  text-align: left;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}
.post-list-btn:hover {
  background-color: #f4f4f0;
}
.post-list-btn.active {
  background-color: var(--accent-light);
  border-color: var(--accent-color);
}
.post-btn-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}
.post-type-tag {
  font-size: 0.65rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  color: var(--text-muted);
}
.post-status-bullet {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}
.post-btn-title {
  font-size: 0.8rem;
  font-weight: 700;
  margin-top: 4px;
}

.contenido-viewer {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.viewer-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.post-main-details h2 {
  font-size: 1.15rem;
  font-weight: 800;
}
.post-meta-text {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.post-action-buttons {
  display: flex;
  gap: 8px;
}

/* CAROUSEL INSTAGRAM FRAME MOCKUP */
.carousel-visual-editor {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.instagram-frame {
  max-width: 300px;
  background-color: #fff;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  overflow: hidden;
  margin: 0 auto;
  box-shadow: var(--shadow-md);
}

.ig-header {
  display: flex;
  align-items: center;
  padding: 10px;
  gap: 8px;
  border-bottom: 1px solid #f4f4f0;
}
.ig-avatar {
  font-size: 1rem;
}
.ig-user-info {
  display: flex;
  flex-direction: column;
}
.ig-username {
  font-weight: 700;
  font-size: 0.75rem;
}
.ig-location {
  font-size: 0.6rem;
  color: var(--text-muted);
}

.ig-slide-body {
  position: relative;
  aspect-ratio: 1;
}
.slide-indicator {
  position: absolute;
  top: 8px;
  right: 8px;
  background-color: rgba(29, 29, 26, 0.8);
  color: #fff;
  font-size: 0.65rem;
  padding: 2px 6px;
  border-radius: 99px;
  font-weight: 700;
}
.slide-designer-canvas {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 24px;
  text-align: center;
}
.slide-canvas-hook {
  font-size: 1.05rem;
  font-weight: 800;
  margin-bottom: 8px;
  letter-spacing: -0.2px;
  color: var(--text-main);
}
.slide-canvas-concept {
  font-size: 0.8rem;
  color: var(--text-muted);
}
.slide-bottom-guide {
  margin-top: auto;
  font-size: 0.6rem;
  color: #a1a19a;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.ig-footer-actions {
  padding: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #f4f4f0;
}
.ig-action-icons {
  display: flex;
  gap: 12px;
  font-size: 1rem;
}
.ig-carousel-dots {
  display: flex;
  gap: 4px;
}
.ig-dot {
  width: 5px;
  height: 5px;
  background-color: #dbdbdb;
  border-radius: 50%;
}
.ig-dot.active {
  background-color: var(--accent-color);
}

.slide-navigator {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 16px;
  margin-top: 16px;
}
.slide-indicator-label {
  font-size: 0.8rem;
  font-weight: 600;
}

/* Feedback panel comments list */
.carousel-feedback-column {
  display: flex;
  flex-direction: column;
}
.feedback-title {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 4px;
}
.feedback-desc {
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-bottom: 12px;
}
.comments-scroller {
  height: 180px;
  overflow-y: auto;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 10px;
  margin-bottom: 12px;
  background-color: #fafaf9;
}
.comment-bubble {
  background-color: #fff;
  border: 1px solid var(--border-color);
  padding: 10px;
  border-radius: var(--radius-sm);
  margin-bottom: 8px;
  box-shadow: var(--shadow-sm);
}
.comment-author {
  font-size: 0.7rem;
  margin-bottom: 2px;
}
.comment-text {
  font-size: 0.75rem;
}

/* REELS Storyboard style */
.reel-details-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

/* Checklist Row Style */
.checklist-item-row {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  background-color: #fff;
  transition: background-color 0.2s;
}
.checklist-item-row.checked {
  background-color: #f0fdf4;
  border-color: #bbf7d0;
}

/* Global utility */
.mt-3 { margin-top: 12px; }
.mt-4 { margin-top: 16px; }
.ml-2 { margin-left: 8px; }
.text-center { text-align: center; }
.w-100 { width: 100%; }
.mt-auto { margin-top: auto; }
</style>
