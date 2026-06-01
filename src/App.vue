<template>
  <div class="app-container">
    
    <!-- Top System Switch Bar: Role Switcher & Branding Personalization Toggle -->
    <div class="system-switch-bar">
      <div class="logo-area">
        <span class="logo-icon">✨</span>
        <span class="logo-text">ContentLab</span>
        <span class="logo-badge">WORKSPACE EDITORIAL</span>
      </div>
      
      <!-- Current Active Role Context Switcher (For prototype testing) -->
      <div class="controls-area">
        <div class="role-selector">
          <span class="selector-icon">👁️</span>
          <select v-model="currentViewMode" class="select-input" @change="onViewModeChange">
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
        <!-- Client Active Brand Card Container (Conditional on Mode) -->
        <div class="sidebar-branding">
          <div class="brand-logo-container">
            <span class="brand-bullet" :style="{ backgroundColor: activeThemeColor }"></span>
            <span class="brand-org-name">{{ selectedBrandingLogo === 'Standard' ? 'ContentLab Studio' : selectedBrandingLogo === 'AgencyAlpha' ? 'Alpha Media Group' : 'Creative Studio' }}</span>
          </div>
        </div>

        <!-- CLIENT SIDEBAR MENU -->
        <nav v-if="currentViewMode === 'client'" class="sidebar-menu client-sidebar">
          <!-- Vista Agencia switcher inside sidebar as shown in references -->
          <div class="sidebar-agency-toggle-container">
            <button @click="currentViewMode = 'agency'" class="vista-agencia-btn">
              🔄 Vista Agencia
            </button>
          </div>

          <div class="menu-section-title">CLIENTE</div>

          <div class="menu-group">
            <button 
              @click="activeClientTab = 'dashboard'" 
              :class="['menu-btn', { active: activeClientTab === 'dashboard' }]"
            >
              <span class="btn-icon">㗊</span> Mi Dashboard
            </button>
            
            <button 
              @click="activeClientTab = 'contenido'" 
              :class="['menu-btn', { active: activeClientTab === 'contenido' }]"
            >
              <span class="btn-icon">🥞</span> Mi Contenido
            </button>
            
            <button 
              @click="activeClientTab = 'sesiones'" 
              :class="['menu-btn', { active: activeClientTab === 'sesiones' }]"
            >
              <span class="btn-icon">📷</span> Sesiones
            </button>

            <button 
              @click="activeClientTab = 'calendario'" 
              :class="['menu-btn', { active: activeClientTab === 'calendario' }]"
            >
              <span class="btn-icon">📅</span> Calendario
            </button>

            <button 
              @click="activeClientTab = 'nube'" 
              :class="['menu-btn', { active: activeClientTab === 'nube' }]"
            >
              <span class="btn-icon">📁</span> Mis Archivos
            </button>

            <button 
              @click="activeClientTab = 'notificaciones'" 
              :class="['menu-btn', { active: activeClientTab === 'notificaciones' }]"
            >
              <span class="btn-icon">🔔</span> Notificaciones
              <span class="badge badge-red" v-if="unreadNotificationsCount > 0">{{ unreadNotificationsCount }}</span>
            </button>
          </div>

          <!-- Active brand footer card exactly as the references -->
          <div class="sidebar-user-footer client-brand-footer mt-auto">
            <div class="user-info">
              <div class="client-avatar-circle">CL</div>
              <div>
                <div class="user-name">Wings &amp; Co.</div>
                <div class="user-role-label">Cliente</div>
              </div>
            </div>
          </div>
        </nav>

        <!-- AGENCY SIDEBAR MENU -->
        <nav v-else class="sidebar-menu">
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
          
          <!-- Clientes Section (Agency View) -->
          <div class="menu-section">
            <div class="menu-sub-header">
              <span class="menu-section-header">CLIENTES</span>
              <span class="ver-todos-link">VER TODOS</span>
            </div>
            
            <div class="client-sublist">
              <div v-for="c in clients" :key="c.id" class="client-sidebar-item-container">
                <button 
                  @click="selectClient(c)"
                  :class="['client-menu-btn', { active: selectedClient?.id === c.id && activeMainMenu === 'client' }]"
                >
                  <span class="client-avatar">{{ c.avatar }}</span>
                  <span class="client-name">{{ c.name }}</span>
                  <span class="chevron-ico">›</span>
                </button>
                
                <!-- Expanded active client sub-tabs menu for Agency side -->
                <div v-if="selectedClient?.id === c.id && activeMainMenu === 'client'" class="active-client-subtabs">
                  <button @click="activeClientTab = 'resumen'" :class="['subtab-link', { active: activeClientTab === 'resumen' }]">
                    <span class="sub-link-icon">📊</span> Resumen
                  </button>
                  <button @click="activeClientTab = 'contenido'" :class="['subtab-link', { active: activeClientTab === 'contenido' }]">
                    <span class="sub-link-icon">👀</span> Estructura
                  </button>
                  <button @click="activeClientTab = 'calendario'" :class="['subtab-link', { active: activeClientTab === 'calendario' }]">
                    <span class="sub-link-icon">📅</span> Calendario
                  </button>
                  <button @click="activeClientTab = 'sesiones'" :class="['subtab-link', { active: activeClientTab === 'sesiones' }]">
                    <span class="sub-link-icon">📸</span> Sesiones
                  </button>
                  <button @click="activeClientTab = 'nube'" :class="['subtab-link', { active: activeClientTab === 'nube' }]">
                    <span class="sub-link-icon">📁</span> Nube
                  </button>
                  <button @click="activeClientTab = 'programacion'" :class="['subtab-link', { active: activeClientTab === 'programacion' }]">
                    <span class="sub-link-icon">🔗</span> Programación
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- Global Strategy Cross-Calendar for PMs only -->
          <div v-if="currentAgencyRole === 'pm'" class="menu-section">
            <div class="menu-section-header">ESTRATEGIA</div>
            <button 
              @click="activeMainMenu = 'global_calendar'" 
              :class="['menu-btn', { active: activeMainMenu === 'global_calendar' }]"
            >
              <span class="btn-icon">📅</span> Calendario Global
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

        <!-- Current User Role Display Footer (Agency view only) -->
        <div v-if="currentViewMode !== 'client'" class="sidebar-user-footer">
          <div class="user-info">
            <div class="user-avatar">👤</div>
            <div>
              <div class="user-name">Alonso Creative</div>
              <div class="user-role-label">
                Agencia: {{ currentAgencyRole.toUpperCase() }}
              </div>
            </div>
          </div>
        </div>
      </aside>

      <!-- CONTENT BODY (Composition Editorial) -->
      <main class="content-body">
        
        <!-- CLIENT MODE VIEW (Híbrido Editorial simple y limpio) -->
        <div v-if="currentViewMode === 'client'" class="client-mode-view">
          
          <!-- Top Bar inside Main View as shown in references -->
          <div class="client-top-bar">
            <button class="icon-btn-toggle" title="Toggle Sidebar">
              🗂️
            </button>
            <span class="client-role-badge">
              <span class="green-dot"></span> Cliente
            </span>
            <div class="top-bar-right">
              <div class="bell-notification-container">
                🔔
                <span class="bell-badge">6</span>
              </div>
            </div>
          </div>

          <!-- CLIENT TAB: MI DASHBOARD -->
          <div v-if="activeClientTab === 'dashboard'" class="client-dashboard-tab animate-fade-in">
            <div class="view-header mb-4">
              <div>
                <h1 class="view-title font-editorial">¡Hola, Wings &amp; Co.! 🍗</h1>
                <p class="view-subtitle">Aquí puedes ver el avance de tu contenido y la evolución de tu cuenta</p>
              </div>
            </div>

            <!-- Notifications Banner Box -->
            <div class="client-banner-card card-blue-notif mb-4">
              <div class="banner-title">
                <span class="bell-icon-notif">🔔</span> Notificaciones
              </div>
              <ul class="banner-list">
                <li>
                  <span>• 2 piezas de contenido están listas para tu revisión y aprobación</span>
                  <button @click="activeClientTab = 'contenido'" class="banner-btn">Aprobar ahora</button>
                </li>
                <li>
                  <span>• La sesión de fotos es el 16 de abril. Revisa los materiales que necesitas tener listos.</span>
                  <button @click="activeClientTab = 'sesiones'" class="banner-btn btn-purple">Ver checklist</button>
                </li>
                <li>
                  <span>• Tu agencia espera tu aprobación en "Nuevas alitas de temporada"</span>
                  <button @click="openPostApproval('post-new-season')" class="banner-btn btn-blue-dark">Revisar contenido</button>
                </li>
              </ul>
            </div>

            <div class="dashboard-editorial-grid">
              <!-- Left Column: Growth & Top Posts -->
              <div class="editorial-main-panel">
                
                <!-- Account Evolution Section -->
                <div class="card mb-4">
                  <h3 class="block-title font-editorial">📊 Evolución de tu cuenta</h3>
                  
                  <div class="stats-row-grid">
                    <div class="stat-box-card">
                      <div class="stat-box-title">
                        <span>👥 Seguidores</span>
                        <span class="trend-indicator up">↗ +12.5%</span>
                      </div>
                      <div class="stat-box-value">4850</div>
                    </div>
                    <div class="stat-box-card">
                      <div class="stat-box-title">
                        <span>❤️ Engagement</span>
                        <span class="trend-indicator up">↗ +0.8pts</span>
                      </div>
                      <div class="stat-box-value">4.2%</div>
                    </div>
                    <div class="stat-box-card">
                      <div class="stat-box-title">
                        <span>👁️ Alcance mensual</span>
                        <span class="trend-indicator up">↗ +23%</span>
                      </div>
                      <div class="stat-box-value">15.200</div>
                    </div>
                    <div class="stat-box-card">
                      <div class="stat-box-title">
                        <span>📈 Impresiones</span>
                        <span class="trend-indicator up">↗ +18%</span>
                      </div>
                      <div class="stat-box-value">28.400</div>
                    </div>
                  </div>
                </div>

                <!-- Follower Growth Line Chart -->
                <div class="card mb-4">
                  <div class="card-header-flex">
                    <h3 class="block-title font-editorial">📈 Crecimiento de seguidores</h3>
                    <span class="chart-timeframe">Últimos 8 meses</span>
                  </div>
                  
                  <!-- CSS Simulated Line Chart -->
                  <div class="simulated-chart-container">
                    <div class="chart-y-axis">
                      <span>5K</span>
                      <span>3K</span>
                      <span>1K</span>
                    </div>
                    <div class="chart-body-wrapper">
                      <!-- Curved line drawing SVG -->
                      <svg class="chart-svg" viewBox="0 0 700 150">
                        <path d="M 20 120 Q 110 90 200 100 T 380 60 T 560 30 T 680 15" fill="none" stroke="var(--accent-color)" stroke-width="3" />
                        <circle cx="20" cy="120" r="4" fill="var(--accent-color)" />
                        <circle cx="110" cy="95" r="4" fill="var(--accent-color)" />
                        <circle cx="200" cy="100" r="4" fill="var(--accent-color)" />
                        <circle cx="290" cy="80" r="4" fill="var(--accent-color)" />
                        <circle cx="380" cy="60" r="4" fill="var(--accent-color)" />
                        <circle cx="470" cy="45" r="4" fill="var(--accent-color)" />
                        <circle cx="560" cy="30" r="4" fill="var(--accent-color)" />
                        <circle cx="680" cy="15" r="4" fill="var(--accent-color)" />
                      </svg>
                      <div class="chart-x-axis">
                        <span>Sep</span>
                        <span>Oct</span>
                        <span>Nov</span>
                        <span>Dic</span>
                        <span>Ene</span>
                        <span>Feb</span>
                        <span>Mar</span>
                        <span>Abr</span>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Best Posts Table -->
                <div class="card mb-4">
                  <h3 class="block-title font-editorial">🏆 Mejores publicaciones</h3>
                  <div class="best-posts-table-wrapper">
                    <table class="best-posts-table">
                      <tbody>
                        <tr>
                          <td class="index-num">1</td>
                          <td class="post-title">¿Cuál es tu sabor favorito?</td>
                          <td class="post-stats-col">
                            <span>❤️ 342</span>
                            <span>💬 89</span>
                            <span>🔗 45</span>
                          </td>
                        </tr>
                        <tr>
                          <td class="index-num">2</td>
                          <td class="post-title">Reseña de cliente</td>
                          <td class="post-stats-col">
                            <span>❤️ 256</span>
                            <span>💬 34</span>
                            <span>🔗 22</span>
                          </td>
                        </tr>
                        <tr>
                          <td class="index-num">3</td>
                          <td class="post-title">Viernes de Wings (anterior)</td>
                          <td class="post-stats-col">
                            <span>❤️ 198</span>
                            <span>💬 56</span>
                            <span>🔗 31</span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>

              </div>

              <!-- Right Column: Weekly Schedule & Awaiting Approvals -->
              <div class="editorial-side-panel">
                
                <!-- This Week Schedule -->
                <div class="card mb-4">
                  <h3 class="block-title font-editorial">🗓️ Esta semana se publicará:</h3>
                  <div class="weekly-schedule-list">
                    <div class="schedule-item">
                      <div class="schedule-day">Miércoles 15</div>
                      <div class="schedule-details">
                        <span class="post-icon-badge color-carousel">📄</span>
                        <div class="schedule-text">
                          <div class="sch-title">¿Cuál es tu sabor favorito?</div>
                          <div class="sch-meta">Producto • Carousel</div>
                        </div>
                        <span class="badge badge-approved">Aprobado</span>
                      </div>
                    </div>

                    <div class="schedule-item">
                      <div class="schedule-day">Viernes 17</div>
                      <div class="schedule-details">
                        <span class="post-icon-badge color-reel">🎥</span>
                        <div class="schedule-text">
                          <div class="sch-title">Viernes de Wings</div>
                          <div class="sch-meta">Lifestyle • Reel</div>
                        </div>
                        <span class="badge badge-process">En proceso</span>
                      </div>
                    </div>

                    <div class="schedule-item">
                      <div class="schedule-day">Martes 21</div>
                      <div class="schedule-details">
                        <span class="post-icon-badge color-reel">🎥</span>
                        <div class="schedule-text">
                          <div class="sch-title">Detrás de la cocina</div>
                          <div class="sch-meta">Momentos de marca • Reel</div>
                        </div>
                        <span class="badge badge-ready">Listo</span>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Awaiting Approvals Card -->
                <div class="card mb-4">
                  <h3 class="block-title font-editorial">⏳ Pendientes de tu aprobación</h3>
                  <div class="awaiting-approval-box">
                    <div class="post-card-preview-header">
                      <span class="post-icon-badge color-carousel">📄 Carousel</span>
                      <h4 class="post-preview-title">Nuevas alitas de temporada</h4>
                    </div>
                    <p class="post-preview-caption">
                      🚨 EDICIÓN LIMITADA 🌶️ 3 sabores nuevos que no puedes perderte. Solo por este mes.
                    </p>
                    
                    <!-- Small interactive mockup view slide preview -->
                    <div class="mini-slide-preview-frame">
                      <div class="mini-slide-content-wrapper">
                        <span class="mini-badge-slide">Slide 1/3</span>
                        <div class="mini-canvas" style="background-color: #fff5f2;">
                          <h5 class="slide-hook">🔥 Nuevas alitas de temporada</h5>
                          <p class="slide-sub">Presentación visual de las salsas de temporada</p>
                        </div>
                      </div>
                    </div>

                    <div class="approval-action-buttons">
                      <button @click="approveActivePostDirectly('post-new-season')" class="btn btn-primary w-100 mb-2">
                        ✅ Aprobar
                      </button>
                      <button @click="requestChangesDirectly('post-new-season')" class="btn btn-secondary w-100">
                        ✏️ Solicitar cambios
                      </button>
                    </div>
                  </div>
                </div>

              </div>
            </div>
          </div>

          <!-- CLIENT TAB: MI CONTENIDO (Pizarra simple cronológica) -->
          <div v-if="activeClientTab === 'contenido'" class="client-contenido-tab animate-fade-in">
            <div class="view-header mb-4">
              <div>
                <h1 class="view-title font-editorial">Mi Contenido</h1>
                <p class="view-subtitle">Revisa la estructura de tu contenido</p>
              </div>
            </div>

            <!-- Grouped Contents Pipeline List -->
            <div class="client-content-timeline">
              <div v-for="g in clientTimelineData" :key="g.date" class="timeline-group-block">
                <div class="timeline-date-label">{{ g.date }}</div>
                
                <div class="timeline-post-card card" @click="openPostDetail(g.post)">
                  <div class="post-card-left">
                    <span :class="['post-type-tag-styled', getPostTypeClass(g.post.type)]">
                      {{ g.post.type }}
                    </span>
                    <div class="post-title-text">{{ g.post.title }}</div>
                    <div class="post-meta-sub">{{ g.post.type }} • {{ g.post.objective }}</div>
                  </div>
                  
                  <div class="post-card-right">
                    <!-- Comment Counter if has comments -->
                    <span v-if="getCommentCount(g.post) > 0" class="comment-count-badge">
                      💬 {{ getCommentCount(g.post) }}
                    </span>
                    <span :class="['badge', getStatusBadgeClass(g.post.status)]">
                      {{ g.post.status }}
                    </span>
                    <span class="arrow-right-ico">➔</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Interactive visual drawer / modal details for client approval & slide comments -->
            <div v-if="clientSelectedDetailPost" class="detail-drawer-overlay" @click.self="clientSelectedDetailPost = null">
              <div class="detail-drawer-body">
                <div class="drawer-header">
                  <div class="drawer-header-meta">
                    <span :class="['badge', getStatusBadgeClass(clientSelectedDetailPost.status)]">{{ clientSelectedDetailPost.status }}</span>
                    <h3>{{ clientSelectedDetailPost.title }}</h3>
                    <p class="drawer-sub">Objetivo: {{ clientSelectedDetailPost.objective }}</p>
                  </div>
                  <button class="btn-close-drawer" @click="clientSelectedDetailPost = null">✕</button>
                </div>

                <div class="drawer-content-grid">
                  <!-- Left side: Visual representation -->
                  <div class="drawer-visual-section">
                    
                    <!-- CAROUSEL MOCKUP -->
                    <div v-if="clientSelectedDetailPost.type === 'Carrusel'" class="carousel-preview-interactive">
                      <div class="instagram-frame frame-centered">
                        <div class="ig-header">
                          <span class="ig-avatar">🍗</span>
                          <div class="ig-user-info">
                            <span class="ig-username">wings_factory</span>
                            <span class="ig-location">Patrocinado</span>
                          </div>
                        </div>

                        <div class="ig-slide-body">
                          <div class="slide-indicator">
                            {{ activeCarouselSlide + 1 }} / {{ clientSelectedDetailPost.slides.length }}
                          </div>
                          
                          <div class="slide-designer-canvas" :style="{ backgroundColor: clientSelectedDetailPost.slides[activeCarouselSlide]?.bgColor || '#fafafa' }">
                            <h4 class="slide-canvas-hook">{{ clientSelectedDetailPost.slides[activeCarouselSlide]?.hook }}</h4>
                            <p class="slide-canvas-concept">{{ clientSelectedDetailPost.slides[activeCarouselSlide]?.concept }}</p>
                            <div class="slide-bottom-guide">[Mockup de Idea Visual]</div>
                          </div>
                        </div>

                        <div class="ig-footer-actions">
                          <div class="ig-action-icons">
                            <span>❤️</span> <span>💬</span> <span>✈️</span>
                          </div>
                          <div class="ig-carousel-dots">
                            <span 
                              v-for="(slide, idx) in clientSelectedDetailPost.slides" 
                              :key="idx" 
                              :class="['ig-dot', { active: idx === activeCarouselSlide }]"
                            ></span>
                          </div>
                        </div>
                      </div>

                      <div class="slide-navigator mt-3">
                        <button @click="prevSlide" :disabled="activeCarouselSlide === 0" class="btn btn-secondary btn-sm">◀ Anterior</button>
                        <span class="slide-indicator-label">Slide {{ activeCarouselSlide + 1 }}</span>
                        <button @click="nextSlide" :disabled="activeCarouselSlide === clientSelectedDetailPost.slides.length - 1" class="btn btn-secondary btn-sm">Siguiente ▶</button>
                      </div>
                    </div>

                    <!-- REEL STORYBOARD -->
                    <div v-else-if="clientSelectedDetailPost.type === 'Reel'" class="reel-preview-interactive">
                      <div class="storyboard-card-scroller">
                        <div v-for="scene in clientSelectedDetailPost.reelDetails.scenes" :key="scene.number" class="storyboard-scene-row card mb-2">
                          <div class="scene-number-title">Escena {{ scene.number }}</div>
                          <p><strong>Visual/Acción:</strong> {{ scene.action }}</p>
                          <p><strong>Audio/Voz en Off:</strong> {{ scene.audio }}</p>
                          <a href="#" class="pin-ref-link" @click.prevent="">📌 Pinterest Ref: {{ scene.ref }}</a>
                        </div>
                      </div>
                    </div>

                    <!-- STORY GRID -->
                    <div v-else-if="clientSelectedDetailPost.type === 'Historia'" class="story-preview-interactive">
                      <div v-for="story in clientSelectedDetailPost.storiesDetails" :key="story.id" class="story-card-body card bg-yellow-story">
                        <div class="story-category-tag">{{ story.category }}</div>
                        <h4 class="story-hook-text">{{ story.hook }}</h4>
                        <p class="story-desc-text">{{ story.concept }}</p>
                      </div>
                    </div>

                  </div>

                  <!-- Right side: Feedback & Actions -->
                  <div class="drawer-feedback-section">
                    
                    <!-- Quick Approvals Switch -->
                    <div class="quick-approval-actions card mb-3">
                      <h4>Flujo de Aprobación</h4>
                      <p class="mb-3 text-muted">¿Está lista esta pieza de contenido para programarse?</p>
                      <div class="action-buttons-flex">
                        <button @click="changePostStatus(clientSelectedDetailPost, 'Aprobado')" class="btn btn-primary w-100 mb-2">
                          ✅ Aprobar Contenido
                        </button>
                        <button @click="changePostStatus(clientSelectedDetailPost, 'En proceso')" class="btn btn-secondary w-100">
                          ✏️ Solicitar Cambios
                        </button>
                      </div>
                    </div>

                    <!-- Comments Contextual Scroller -->
                    <div class="drawer-comments-box card">
                      <h4>💬 Observaciones</h4>
                      <p class="text-muted mb-2">Escribe tus comentarios para el equipo creativo de la agencia.</p>
                      
                      <div class="drawer-comments-list">
                        <div 
                          v-for="comment in getPostComments(clientSelectedDetailPost)" 
                          :key="comment.id" 
                          class="comment-bubble-item"
                        >
                          <div class="comment-bubble-header">
                            <strong>{{ comment.author }}</strong>
                            <span class="comment-badge">{{ comment.role }}</span>
                          </div>
                          <p class="comment-bubble-text">{{ comment.text }}</p>
                        </div>
                      </div>

                      <div class="comment-input-row mt-2">
                        <input 
                          type="text" 
                          v-model="clientNewCommentText" 
                          placeholder="Añade un comentario..." 
                          class="form-control"
                          @keyup.enter="addClientComment"
                        />
                        <button @click="addClientComment" class="btn btn-primary btn-sm mt-2">Enviar</button>
                      </div>
                    </div>

                  </div>
                </div>
              </div>
            </div>

          </div>

          <!-- CLIENT TAB: SESIONES (Visual Cards matching mock reference) -->
          <div v-if="activeClientTab === 'sesiones'" class="client-sesiones-tab animate-fade-in">
            <div class="view-header mb-4">
              <div>
                <h1 class="view-title font-editorial">Sesiones Programadas</h1>
                <p class="view-subtitle">Revisa qué necesitas tener listo para cada sesión</p>
              </div>
            </div>

            <div class="sessions-section-title mb-3">
              <span>📷 📸 Sesiones de Fotos</span>
              <span class="sessions-count-badge">2</span>
            </div>

            <!-- Single Expandable Session Card matching screenshot layout -->
            <div class="session-main-card card">
              <div class="session-card-header-row" @click="photoSessionOpen = !photoSessionOpen">
                <div class="header-left">
                  <span class="camera-icon-circle">📷</span>
                  <div class="header-titles">
                    <h3 class="session-title-text font-editorial">Foto 1 — ¿Cuál es tu sabor favorito?</h3>
                    <p class="session-subtitle-text">📷 Sesión de fotos • Estudio / Restaurante Wings &amp; Co.</p>
                  </div>
                </div>
                <div class="header-right">
                  <span class="badge badge-approved">Confirmada</span>
                  <span class="dropdown-arrow-icon">{{ photoSessionOpen ? '▲' : '▼' }}</span>
                </div>
              </div>

              <!-- Collapsible Details Block -->
              <div v-if="photoSessionOpen" class="session-card-body-details">
                
                <!-- Quick Specs Row -->
                <div class="session-specs-row">
                  <div class="spec-col-item">
                    <span class="spec-icon">📍</span>
                    <div>
                      <div class="spec-label">UBICACIÓN</div>
                      <div class="spec-value">Estudio / Restaurante Wings &amp; Co.</div>
                    </div>
                  </div>
                  <div class="spec-col-item">
                    <span class="spec-icon">⏱️</span>
                    <div>
                      <div class="spec-label">DURACIÓN</div>
                      <div class="spec-value">2 horas</div>
                    </div>
                  </div>
                  <div class="spec-col-item">
                    <span class="spec-icon">👥</span>
                    <div>
                      <div class="spec-label">PERSONAS</div>
                      <div class="spec-value">2 rol(es)</div>
                    </div>
                  </div>
                </div>

                <!-- Personas y Vestimenta -->
                <div class="personas-vestimenta-block mt-4">
                  <h4 class="section-divider-title">👤 Personas y Vestimenta</h4>
                  
                  <div class="roles-cards-grid">
                    <!-- Role 1 -->
                    <div class="role-detail-card">
                      <div class="role-pill-header">
                        <span class="role-pill-badge">Modelo principal</span>
                      </div>
                      <h4 class="role-desc-title">Hombre joven 20-30 años, look casual</h4>
                      
                      <ul class="role-bullet-list">
                        <li>
                          <span class="bullet-ico">👕</span>
                          <span>Playera negra, jeans, tenis blancos</span>
                        </li>
                        <li>
                          <span class="bullet-ico">🔗</span>
                          <a href="https://pinterest.com/pin/modelo-casual-hombre" target="_blank" class="pinterest-link-style">
                            pinterest.com/pin/modelo-casual-hombre
                          </a>
                        </li>
                        <li>
                          <span class="bullet-ico">💡</span>
                          <span class="italic-tip">Sonrisa natural, que se vea disfrutando</span>
                        </li>
                      </ul>
                    </div>

                    <!-- Role 2 -->
                    <div class="role-detail-card">
                      <div class="role-pill-header">
                        <span class="role-pill-badge">Modelo secundario</span>
                      </div>
                      <h4 class="role-desc-title">Mujer joven 20-28 años</h4>
                      
                      <ul class="role-bullet-list">
                        <li>
                          <span class="bullet-ico">👕</span>
                          <span>Outfit colorido, accesorios llamativos</span>
                        </li>
                        <li>
                          <span class="bullet-ico">🔗</span>
                          <a href="https://pinterest.com/pin/look-colorido-mujer" target="_blank" class="pinterest-link-style">
                            pinterest.com/pin/look-colorido-mujer
                          </a>
                        </li>
                        <li>
                          <span class="bullet-ico">💡</span>
                          <span class="italic-tip">Expresiones de sorpresa/antojo</span>
                        </li>
                      </ul>
                    </div>
                  </div>
                </div>

                <!-- Bottom References Row -->
                <div class="session-bottom-refs-row mt-4">
                  <div class="ref-link-card">
                    <span class="ref-icon">👚</span> Refs. de Vestimenta
                  </div>
                  <div class="ref-link-card">
                    <span class="ref-icon">🖼️</span> Refs. de Persona Ideal
                  </div>
                </div>

              </div>
            </div>

            <!-- Client Responsibilities Checklist (Preparativos en tienda) -->
            <div class="card checklist-shop-card mt-4">
              <h3 class="col-title-primary font-editorial">🏪 Responsabilidad de Tienda (Listado de Preparativos)</h3>
              <p class="col-subtitle">Lo que debes tener listo antes de que comience la sesión en el local.</p>
              
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

          <!-- CLIENT TAB: CALENDARIO (Exact layout mapping monthly calendar grid) -->
          <div v-if="activeClientTab === 'calendario'" class="client-calendario-tab animate-fade-in">
            <div class="view-header mb-4">
              <div>
                <h1 class="view-title font-editorial">Calendario de Contenido</h1>
                <p class="view-subtitle">Vista mensual de publicaciones programadas</p>
              </div>
            </div>

            <!-- Filter Categories Row -->
            <div class="calendar-categories-filters mb-4">
              <button class="filter-category-btn active-carrusel">📄 Carrusel</button>
              <button class="filter-category-btn active-reel">🎥 Reel</button>
              <button class="filter-category-btn active-story">📱 Historia</button>
              <button class="filter-category-btn active-post">🖼️ Post</button>
            </div>

            <!-- Calendar Grid Frame -->
            <div class="card calendar-grid-card">
              <!-- Calendar Navigation Header -->
              <div class="calendar-nav-row">
                <button class="nav-arrow-btn">◀</button>
                <h2 class="month-title font-editorial">Abril de 2026</h2>
                <button class="nav-arrow-btn">▶</button>
              </div>

              <div class="calendar-table-container">
                <table class="calendar-table-grid">
                  <thead>
                    <tr>
                      <th>Lun</th>
                      <th>Mar</th>
                      <th>Mié</th>
                      <th>Jue</th>
                      <th>Vie</th>
                      <th>Sáb</th>
                      <th>Dom</th>
                    </tr>
                  </thead>
                  <tbody>
                    <!-- Row 1 -->
                    <tr>
                      <td class="disabled-day"></td>
                      <td class="disabled-day"></td>
                      <td><div class="day-number">1</div></td>
                      <td><div class="day-number">2</div></td>
                      <td><div class="day-number">3</div></td>
                      <td><div class="day-number">4</div></td>
                      <td><div class="day-number">5</div></td>
                    </tr>
                    <!-- Row 2 -->
                    <tr>
                      <td><div class="day-number">6</div></td>
                      <td><div class="day-number">7</div></td>
                      <td><div class="day-number">8</div></td>
                      <td><div class="day-number">9</div></td>
                      <td>
                        <div class="day-number">10</div>
                        <div class="calendar-event-tag tag-story">Reseña de cliente</div>
                      </td>
                      <td><div class="day-number">11</div></td>
                      <td><div class="day-number">12</div></td>
                    </tr>
                    <!-- Row 3 -->
                    <tr>
                      <td class="today-highlight">
                        <div class="day-number circle-blue">13</div>
                      </td>
                      <td><div class="day-number">14</div></td>
                      <td>
                        <div class="day-number">15</div>
                        <div class="calendar-event-tag tag-carousel">¿Cuál es tu sabor favorito?</div>
                      </td>
                      <td><div class="day-number">16</div></td>
                      <td>
                        <div class="day-number">17</div>
                        <div class="calendar-event-tag tag-reel">Viernes de Wings</div>
                      </td>
                      <td><div class="day-number">18</div></td>
                      <td>
                        <div class="day-number">19</div>
                        <div class="calendar-event-tag tag-post">Combo Amigos 2x1</div>
                      </td>
                    </tr>
                    <!-- Row 4 -->
                    <tr>
                      <td><div class="day-number">20</div></td>
                      <td>
                        <div class="day-number">21</div>
                        <div class="calendar-event-tag tag-reel">Detrás de la cocina</div>
                      </td>
                      <td>
                        <div class="day-number">22</div>
                        <div class="calendar-event-tag tag-carousel">Nuevas alitas de temporada</div>
                      </td>
                      <td><div class="day-number">23</div></td>
                      <td><div class="day-number">24</div></td>
                      <td><div class="day-number">25</div></td>
                      <td><div class="day-number">26</div></td>
                    </tr>
                    <!-- Row 5 -->
                    <tr>
                      <td><div class="day-number">27</div></td>
                      <td><div class="day-number">28</div></td>
                      <td><div class="day-number">29</div></td>
                      <td><div class="day-number">30</div></td>
                      <td class="disabled-day"></td>
                      <td class="disabled-day"></td>
                      <td class="disabled-day"></td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>

          <!-- CLIENT TAB: MIS ARCHIVOS -->
          <div v-if="activeClientTab === 'nube'" class="client-nube-tab animate-fade-in">
            <div class="view-header mb-4">
              <div>
                <h1 class="view-title font-editorial">Mis Archivos</h1>
                <p class="view-subtitle">Organización interna de archivos por cliente y mes</p>
              </div>
            </div>

            <div class="card cloud-storage-card">
              <div class="cloud-header">
                <h3>📁 Almacenamiento Nube</h3>
                <p class="cloud-desc">Accede a las referencias, videos y fotos finales cargadas.</p>
              </div>

              <div class="cloud-folder-tree">
                <div class="folder-row">
                  <span class="folder-icon">📂</span>
                  <strong>Abril 2026</strong>
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

          <!-- CLIENT TAB: NOTIFICACIONES -->
          <div v-if="activeClientTab === 'notificaciones'" class="client-notificaciones-tab animate-fade-in">
            <div class="view-header mb-4">
              <div>
                <h1 class="view-title font-editorial">Notificaciones</h1>
                <p class="view-subtitle">Historial de comentarios y novedades en tus publicaciones</p>
              </div>
              <button @click="markAllNotificationsAsRead" class="btn btn-secondary btn-sm">Marcar todas como leídas</button>
            </div>

            <div class="card">
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

        <!-- AGENCY MODE VIEW -->
        <div v-else class="agency-mode-view">
          
          <!-- MODULE: DASHBOARD -->
          <div v-if="activeMainMenu === 'dashboard'" class="module-view">
            <div class="view-header">
              <div>
                <span class="view-eyebrow">RESUMEN CREATIVO</span>
                <h1 class="view-title font-editorial">Planificación de la Semana</h1>
                <p class="view-subtitle">Foco central en aprobaciones de contenido, storyboards y rodajes activos.</p>
              </div>
              <div class="header-actions">
                <span class="metric-summary-pill">Procesando: {{ activePostsCount }} piezas en pipeline</span>
              </div>
            </div>

            <!-- EDITORIAL DASHBOARD COMPOSITION -->
            <div class="dashboard-editorial-grid">
              
              <!-- Left Side: Editorial Core Focus -->
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
                          <strong>¿Cuál es tu sabor favorito?</strong>
                          <p class="desc-text">[Mockup: Alitas crujientes con humo animado]</p>
                        </div>
                      </div>
                      <div class="mini-ig-footer">
                        <span>💬 1 Observación activa</span>
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

                <!-- Composition list of active processes -->
                <div class="editorial-list-block card">
                  <h3 class="block-title font-editorial">Aprobaciones y Flujo Creativo Reciente</h3>
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

              <!-- Right Side: Secondary Column -->
              <div class="editorial-side-panel">
                <div class="card simple-kpi-card mb-4">
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
                    <span class="tag-status">Abril 2026</span>
                  </div>
                  <p class="desc">Preparativos requeridos en tienda física:</p>
                  <ul class="mini-checklist">
                    <li v-for="item in storeChecklist" :key="item.id">
                      <input type="checkbox" v-model="item.completed" />
                      <span>{{ item.label.substring(0, 40) }}...</span>
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

              <!-- AI Output pillars display inside brief new -->
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

          <!-- MODULE: REPOSITORIO DE MARCA (DATABASE) -->
          <div v-if="activeMainMenu === 'database'" class="module-view">
            <div class="view-header">
              <div>
                <h1 class="view-title font-editorial">Base de datos</h1>
                <p class="view-subtitle">Repositorio central de información de marcas, briefs e historial</p>
              </div>
            </div>

            <div class="search-bar-container mt-2">
              <input type="text" placeholder="🔍 Buscar en la base de datos..." class="form-control w-100" />
            </div>

            <div class="grid-repo mt-3">
              <div class="card repo-stat-card">
                <div class="stat-header">
                  <span class="stat-icon-circle">👥</span>
                  <span class="stat-badge-light">4 registros</span>
                </div>
                <h3 class="stat-title-custom font-editorial">Clientes</h3>
                <p class="stat-desc-custom">Información, planes y redes sociales</p>
              </div>

              <div class="card repo-stat-card">
                <div class="stat-header">
                  <span class="stat-icon-circle">📄</span>
                  <span class="stat-badge-light">1 registros</span>
                </div>
                <h3 class="stat-title-custom font-editorial">Briefs</h3>
                <p class="stat-desc-custom">Briefs históricos y pilares por marca</p>
              </div>

              <div class="card repo-stat-card">
                <div class="stat-header">
                  <span class="stat-icon-circle">🏷️</span>
                  <span class="stat-badge-light">5 registros</span>
                </div>
                <h3 class="stat-title-custom font-editorial">Pilares de contenido</h3>
                <p class="stat-desc-custom">Distribución temática por marca</p>
              </div>

              <div class="card repo-stat-card">
                <div class="stat-header">
                  <span class="stat-icon-circle">📦</span>
                  <span class="stat-badge-light">24 registros</span>
                </div>
                <h3 class="stat-title-custom font-editorial">Activos y referencias</h3>
                <p class="stat-desc-custom">Banco de referencias visuales y guiones</p>
              </div>
            </div>

            <div class="card mt-4">
              <h3 class="block-title font-editorial">Marcas registradas</h3>
              <div class="registered-brands-list">
                <div class="brand-row-item">
                  <div class="brand-info-block">
                    <span class="brand-avatar-big">🍗</span>
                    <div>
                      <strong>Wings &amp; Co.</strong>
                      <p>Restaurante de Alitas · Instagram, Facebook, TikTok</p>
                    </div>
                  </div>
                  <span class="brand-tier-label">Premium</span>
                </div>

                <div class="brand-row-item">
                  <div class="brand-info-block">
                    <span class="brand-avatar-big">💪</span>
                    <div>
                      <strong>FitZone Gym</strong>
                      <p>Fitness · Instagram, TikTok</p>
                    </div>
                  </div>
                  <span class="brand-tier-label label-basic">Básico</span>
                </div>

                <div class="brand-row-item">
                  <div class="brand-info-block">
                    <span class="brand-avatar-big">☕</span>
                    <div>
                      <strong>Café Aroma</strong>
                      <p>Cafetería · Instagram, Facebook</p>
                    </div>
                  </div>
                  <span class="brand-tier-label">Premium</span>
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

          <!-- MODULE: GLOBAL CALENDAR (CROSS EDITORIAL CALENDAR) -->
          <div v-if="activeMainMenu === 'global_calendar'" class="module-view">
            <div class="view-header">
              <div>
                <h1 class="view-title font-editorial">📅 Calendario global de contenido</h1>
                <p class="view-subtitle">Programación cruzada de todas las marcas · detección automática de carga para diseño</p>
              </div>
            </div>

            <div class="editorial-timeline-list">
              
              <div class="timeline-day-card card">
                <div class="timeline-day-sidebar">
                  <span class="day-number-bubble">JUE 9</span>
                  <p class="day-details">1 pieza · 1 marca</p>
                </div>
                <div class="timeline-day-contents">
                  <div class="timeline-post-row">
                    <span class="post-brand-ico">🍗</span>
                    <div class="post-text-details">
                      <span class="post-type-tag">Historia</span>
                      <strong>Reseña de cliente</strong>
                      <p>Wings &amp; Co. · UGC / Comunidad</p>
                    </div>
                    <span class="post-type-pill text-story">STORY</span>
                  </div>
                </div>
              </div>

              <div class="timeline-day-card card mt-3">
                <div class="timeline-day-sidebar">
                  <span class="day-number-bubble">MAR 14</span>
                  <p class="day-details">1 pieza · 1 marca</p>
                </div>
                <div class="timeline-day-contents">
                  <div class="timeline-post-row">
                    <span class="post-brand-ico">🍗</span>
                    <div class="post-text-details">
                      <span class="post-type-tag">Carrusel</span>
                      <strong>¿Cuál es tu sabor favorito?</strong>
                      <p>Wings &amp; Co. · Producto</p>
                    </div>
                    <span class="post-type-pill text-carousel-badge">CAROUSEL</span>
                  </div>
                </div>
              </div>

              <div class="timeline-day-card card mt-3">
                <div class="timeline-day-sidebar">
                  <span class="day-number-bubble">JUE 16</span>
                  <p class="day-details">1 pieza · 1 marca</p>
                </div>
                <div class="timeline-day-contents">
                  <div class="timeline-post-row">
                    <span class="post-brand-ico">🍗</span>
                    <div class="post-text-details">
                      <span class="post-type-tag">Reel</span>
                      <strong>Viernes de Wings</strong>
                      <p>Wings &amp; Co. · Lifestyle</p>
                    </div>
                    <span class="post-type-pill text-reel-badge">REEL</span>
                  </div>
                </div>
              </div>

              <div class="timeline-day-card card mt-3">
                <div class="timeline-day-sidebar">
                  <span class="day-number-bubble">SÁB 18</span>
                  <p class="day-details">1 pieza · 1 marca</p>
                </div>
                <div class="timeline-day-contents">
                  <div class="timeline-post-row">
                    <span class="post-brand-ico">🍗</span>
                    <div class="post-text-details">
                      <span class="post-type-tag">Post</span>
                      <strong>Combo Amigos 2x1</strong>
                      <p>Wings &amp; Co. · Promociones</p>
                    </div>
                    <span class="post-type-pill text-post-badge">POST</span>
                  </div>
                </div>
              </div>

            </div>
          </div>

          <!-- MODULE: CLIENT SPECIFIC VIEW (Agency side Client workspace detail) -->
          <div v-if="activeMainMenu === 'client' && selectedClient" class="module-view">
            <div class="client-detail-header">
              <div class="client-logo-title">
                <span class="client-detail-avatar">{{ selectedClient.avatar }}</span>
                <div>
                  <h1 class="client-detail-name font-editorial">{{ selectedClient.name }}</h1>
                  <p class="client-detail-niche">Rubro: {{ selectedClient.niche }}</p>
                </div>
              </div>

              <div class="client-header-actions">
                <button @click="downloadClientPDF" class="btn btn-secondary">
                  📥 Descargar Estructuras PDF
                </button>
              </div>
            </div>

            <!-- Client Submenu Tabs (Agency side view of client details) -->
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

            <!-- AGENCY VIEW SUBTAB: RESUMEN -->
            <div v-if="activeClientTab === 'resumen'" class="subtab-view mt-3">
              <div class="grid-resumen">
                <div class="card">
                  <h3 class="card-title font-editorial">Resumen de Contrato</h3>
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
                    <h3 class="card-title font-editorial">📈 Evolución Visual de Marca</h3>
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

            <!-- AGENCY VIEW SUBTAB: CONTENIDO (CORE EDITOR PIZARRA) -->
            <div v-if="activeClientTab === 'contenido'" class="subtab-view mt-3">
              <div class="contenido-grid">
                
                <!-- Left Column: Content list selector -->
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
                      <h2 class="font-editorial">{{ currentPost.title }}</h2>
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

                  <!-- Creative Asset Uploader (Exclusivo Agencia) -->
                  <div class="card uploader-card mt-4">
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

            <!-- AGENCY VIEW SUBTAB: CALENDARIO -->
            <div v-if="activeClientTab === 'calendario'" class="subtab-view mt-3">
              <div class="card">
                <div class="calendar-controls">
                  <h3 class="font-editorial">Calendario de Contenidos - {{ selectedClient.name }}</h3>
                  <div class="calendar-legend">
                    <span class="leg-item"><span class="bullet" style="background-color: var(--status-idea)"></span> Idea</span>
                    <span class="leg-item"><span class="bullet" style="background-color: var(--status-process)"></span> En proceso</span>
                    <span class="leg-item"><span class="bullet" style="background-color: var(--status-ready)"></span> Listo</span>
                    <span class="leg-item"><span class="bullet" style="background-color: var(--status-approved)"></span> Aprobado</span>
                  </div>
                </div>

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

            <!-- AGENCY VIEW SUBTAB: SESIONES DE FOTO Y VIDEO -->
            <div v-if="activeClientTab === 'sesiones'" class="subtab-view mt-3">
              <div class="sesiones-layout">
                
                <!-- Photos Grouping -->
                <div class="card sesion-col">
                  <h3 class="col-title-primary font-editorial">📸 Sesión de Fotos Planificadas</h3>
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
                  <h3 class="col-title-primary font-editorial">🎥 Sesión de Videos Planificadas</h3>
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

                <!-- Responsabilidad de Tienda checklist -->
                <div class="card checklist-shop-card">
                  <h3 class="col-title-primary font-editorial">🏪 Responsabilidad de Tienda (Listado de Preparativos)</h3>
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

            <!-- AGENCY VIEW SUBTAB: NUBE -->
            <div v-if="activeClientTab === 'nube'" class="subtab-view mt-3">
              <div class="card cloud-storage-card">
                <div class="cloud-header">
                  <h3>📁 Almacenamiento Nube</h3>
                  <p class="cloud-desc">Organización interna de archivos por cliente y mes.</p>
                </div>

                <div class="cloud-folder-tree">
                  <div class="folder-row">
                    <span class="folder-icon">📂</span>
                    <strong>Abril 2026</strong>
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

            <!-- AGENCY VIEW SUBTAB: PROGRAMACIÓN -->
            <div v-if="activeClientTab === 'programacion'" class="subtab-view mt-3">
              <div class="card text-center">
                <h3>🔗 Conexión &amp; Programación de Redes</h3>
                <p>Simulador de calendarización automática de posts en plataformas conectadas.</p>
                
                <div class="mt-4">
                  <span class="badge badge-approved">Instagram Conectado</span>
                  <span class="badge badge-idea ml-2">TikTok (Pendiente)</span>
                </div>
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
      activeClientTab: 'dashboard', // default to dashboard tab
      activeBriefTab: 'new', // new, history
      
      // Toggle for session details in client view
      photoSessionOpen: true,

      // Selected brand context
      selectedClient: null,
      activePostId: null,
      activeCarouselSlide: 0,
      
      // Comments drawers/panels
      clientSelectedDetailPost: null,
      clientNewCommentText: '',
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

      briefsHistory: [
        {
          id: 'brief-1',
          clientName: 'Wings Factory',
          niche: 'restaurante',
          goal: 'Establecer la calidad premium y el sabor crujiente de las alitas artesanales.',
          monthlyPosts: 12,
          tone: 'divertido',
          date: '15/04/2026',
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
          name: 'Wings & Co.',
          avatar: '🍗',
          niche: 'Restaurante de Alitas',
          posts: [
            {
              id: 'post-1',
              type: 'Carrusel',
              title: '¿Cuál es tu sabor favorito?',
              date: 'Miércoles 15',
              objective: 'Producto',
              status: 'Aprobado',
              slides: [
                {
                  number: 1,
                  hook: '🔥 ¿Cuál es tu sabor favorito?',
                  concept: 'Mockup visual con imagen gigante de alitas crujientes bañadas en salsas con humo animado.',
                  bgColor: '#fff8f6',
                  comments: [
                    { id: 1, author: 'Carlos (Supervisor PM)', role: 'PM', text: 'Me gusta el hook, asegúrate que las alitas se vean súper jugosas.' }
                  ]
                },
                {
                  number: 2,
                  hook: '🥵 ¿Te atreves con el Mango Habanero?',
                  concept: 'Detalle de alitas picantes con rodajas de chile habanero fresco.',
                  bgColor: '#fffcf5',
                  comments: []
                },
                {
                  number: 3,
                  hook: '💀 Apocalypse Pepper: Nivel Máximo',
                  concept: 'Advertencia en letras grandes rojas y primer plano de la salsa apocalíptica.',
                  bgColor: '#fff5f2',
                  comments: []
                }
              ]
            },
            {
              id: 'post-2',
              type: 'Reel',
              title: 'Viernes de Wings',
              date: 'Viernes 17',
              objective: 'Lifestyle',
              status: 'En proceso',
              reelDetails: {
                acting: 'Grupo de amigos compartiendo una bandeja gigante de alitas. Risas y primeros planos de las salsas.',
                voiceover: 'Voz en off: El fin de semana sabe mejor cuando se comparte. ¿Listos para el viernes de wings?',
                pinterestRef: 'pinterest.com/pin/viernes-wings-lifestyle',
                scenes: [
                  { number: 1, action: 'Entrada de amigos al local saludando a la cámara.', audio: 'Música alegre de fondo.', ref: 'lifestyle-1' },
                  { number: 2, action: 'Primer plano de alitas crujientes chorreando salsa.', audio: 'Sonido de mordisco.', ref: 'lifestyle-2' }
                ]
              }
            },
            {
              id: 'post-3',
              type: 'Historia',
              title: 'Reseña de cliente',
              date: 'Viernes 10',
              objective: 'UGC / Comunidad',
              status: 'Publicado',
              storiesDetails: [
                { id: 'st-1', category: 'UGC', hook: 'Lo que nuestros clientes opinan de nosotros ⭐⭐⭐⭐⭐', concept: 'Repost de una historia de cliente comiendo alitas' }
              ]
            },
            {
              id: 'post-4',
              type: 'Post',
              title: 'Combo Amigos 2x1',
              date: 'Domingo 19',
              objective: 'Promociones',
              status: 'Idea'
            },
            {
              id: 'post-5',
              type: 'Reel',
              title: 'Detrás de la cocina',
              date: 'Martes 21',
              objective: 'Momentos de marca',
              status: 'Listo',
              reelDetails: {
                acting: 'Cocinero empanizando alitas y friéndolas al momento para mostrar la frescura.',
                voiceover: 'Música rápida e instrumental que transmite energía.',
                pinterestRef: 'pinterest.com/pin/detras-cocina-wings',
                scenes: [
                  { number: 1, action: 'Preparación de las alitas con las especias secretas.', audio: 'Efecto de sonido de batido.', ref: 'cocina-1' },
                  { number: 2, action: 'Saliendo calientes y crujientes de la freidora.', audio: 'Sonido crujiente.', ref: 'cocina-2' }
                ]
              }
            },
            {
              id: 'post-new-season',
              type: 'Carrusel',
              title: 'Nuevas alitas de temporada',
              date: 'Miércoles 22',
              objective: 'Lanzamiento',
              status: 'En revisión',
              slides: [
                {
                  number: 1,
                  hook: '🔥 Nuevas alitas de temporada',
                  concept: 'Diseño en colores oscuros destacando los 3 sabores nuevos.',
                  bgColor: '#fff5f2',
                  comments: []
                }
              ]
            }
          ]
        },
        {
          id: 'client-2',
          name: 'FitZone Gym',
          avatar: '💪',
          niche: 'Gimnasio & Fitness',
          posts: []
        },
        {
          id: 'client-3',
          name: 'Café Aroma',
          avatar: '☕',
          niche: 'Cafetería de Especialidad',
          posts: []
        }
      ],

      storeChecklist: [
        { id: 1, label: 'Tener preparadas 10 alitas crujientes recién bañadas para la toma de producto a las 4 PM', completed: false },
        { id: 2, label: 'Limpiar mesa rústica de madera cerca del ventanal para luz natural', completed: true },
        { id: 3, label: 'Contar con envases corporativos limpios y listos', completed: false }
      ],

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
        return { '--accent-color': '#f97316', '--accent-hover': '#ea580c' };
      }
      if (this.selectedBrandingLogo === 'CreativeStudio') {
        return { '--accent-color': '#8b5cf6', '--accent-hover': '#7c3aed' };
      }
      return { '--accent-color': '#0d9488', '--accent-hover': '#0f766e' };
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
    },
    clientTimelineData() {
      const activeClient = this.clients[0]; // Wings & Co.
      if (!activeClient || !activeClient.posts) return [];
      
      // Return sorted timeline mapping
      return activeClient.posts.map(p => ({
        date: p.date,
        post: p
      })).sort((a, b) => {
        const getDayVal = dateStr => {
          if (dateStr.includes('10')) return 10;
          if (dateStr.includes('15')) return 15;
          if (dateStr.includes('17')) return 17;
          if (dateStr.includes('19')) return 19;
          if (dateStr.includes('21')) return 21;
          if (dateStr.includes('22')) return 22;
          return 30;
        };
        return getDayVal(a.date) - getDayVal(b.date);
      });
    }
  },
  methods: {
    onViewModeChange() {
      if (this.currentViewMode === 'client') {
        this.selectedClient = this.clients[0];
        this.activeClientTab = 'dashboard';
      } else {
        this.activeMainMenu = 'dashboard';
        this.activeClientTab = 'resumen';
      }
    },
    selectClient(client) {
      this.selectedClient = client;
      this.activeMainMenu = 'client';
      this.activeClientTab = 'resumen';
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
      const targetPost = this.currentViewMode === 'client' ? this.clientSelectedDetailPost : this.currentPost;
      if (targetPost && this.activeCarouselSlide < targetPost.slides.length - 1) {
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
          author: 'Agencia (' + this.currentAgencyRole.toUpperCase() + ')',
          role: this.currentAgencyRole.toUpperCase(),
          text: this.newCommentText
        });
        this.newCommentText = '';
      }
    },
    addClientComment() {
      if (!this.clientNewCommentText.trim() || !this.clientSelectedDetailPost) return;
      
      const commentObj = {
        id: Date.now(),
        author: 'Wings & Co.',
        role: 'Cliente',
        text: this.clientNewCommentText
      };
      
      if (this.clientSelectedDetailPost.type === 'Carrusel') {
        const slide = this.clientSelectedDetailPost.slides[this.activeCarouselSlide];
        if (slide) {
          if (!slide.comments) slide.comments = [];
          slide.comments.push(commentObj);
        }
      } else {
        if (!this.clientSelectedDetailPost.generalComments) {
          this.clientSelectedDetailPost.generalComments = [];
        }
        this.clientSelectedDetailPost.generalComments.push(commentObj);
      }
      
      this.notifications.unshift({
        id: Date.now(),
        type: 'client',
        msg: `Cliente dejó un comentario en "${this.clientSelectedDetailPost.title}"`,
        time: 'Ahora mismo',
        read: false
      });
      
      this.clientNewCommentText = '';
    },
    getPostComments(post) {
      if (!post) return [];
      if (post.type === 'Carrusel') {
        const slide = post.slides[this.activeCarouselSlide];
        return slide ? (slide.comments || []) : [];
      }
      return post.generalComments || [];
    },
    getCommentCount(post) {
      if (!post) return 0;
      if (post.type === 'Carrusel') {
        return post.slides.reduce((acc, s) => acc + (s.comments ? s.comments.length : 0), 0);
      }
      return post.generalComments ? post.generalComments.length : 0;
    },
    changePostStatus(post, newStatus) {
      post.status = newStatus;
      this.notifications.unshift({
        id: Date.now(),
        type: 'client',
        msg: `El post "${post.title}" se cambió a estado: ${newStatus}`,
        time: 'Hace un momento',
        read: false
      });
    },
    approveActivePostDirectly(postId) {
      const activeClient = this.clients[0];
      const post = activeClient.posts.find(p => p.id === postId);
      if (post) {
        post.status = 'Aprobado';
        alert(`¡Estructura "${post.title}" Aprobada con éxito!`);
      }
    },
    requestChangesDirectly(postId) {
      const activeClient = this.clients[0];
      const post = activeClient.posts.find(p => p.id === postId);
      if (post) {
        post.status = 'En proceso';
        alert(`Se han solicitado cambios para "${post.title}" al equipo creativo.`);
      }
    },
    openPostApproval(postId) {
      const activeClient = this.clients[0];
      const post = activeClient.posts.find(p => p.id === postId);
      if (post) {
        this.activeClientTab = 'contenido';
        this.openPostDetail(post);
      }
    },
    openPostDetail(post) {
      this.clientSelectedDetailPost = post;
      this.activeCarouselSlide = 0;
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
        posts: []
      });

      alert('¡Brief guardado con éxito! Se ha creado una carpeta de cliente independiente.');
      this.activeBriefTab = 'history';
    },
    downloadClientPDF() {
      alert('📄 PDF Descargado: Se descargó el dossier estructurado con aprobaciones y mockups.');
    },
    markAllNotificationsAsRead() {
      this.notifications.forEach(n => n.read = true);
    },
    getPostTypeClass(type) {
      if (type === 'Carrusel') return 'color-carousel-tag';
      if (type === 'Reel') return 'color-reel-tag';
      if (type === 'Historia') return 'color-story-tag';
      return 'color-post-tag';
    },
    getStatusBadgeClass(status) {
      if (status === 'Publicado') return 'badge-published';
      if (status === 'Aprobado') return 'badge-approved';
      if (status === 'En proceso') return 'badge-process';
      if (status === 'Listo') return 'badge-ready';
      return 'badge-idea';
    }
  },
  mounted() {
    if (this.clients.length > 0) {
      this.selectedClient = this.clients[0];
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

/* Sidebar styling */
.sidebar {
  width: 230px;
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

/* Vista Agencia Sidebar Button */
.sidebar-agency-toggle-container {
  padding: 4px 6px;
}
.vista-agencia-btn {
  width: 100%;
  padding: 10px;
  border: none;
  background-color: #ede9fe;
  color: #5b21b6;
  font-weight: 700;
  font-size: 0.8rem;
  border-radius: var(--radius-md);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  transition: background-color 0.2s;
}
.vista-agencia-btn:hover {
  background-color: #ddd6fe;
}

.menu-section-title {
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  color: #a1a19a;
  letter-spacing: 0.5px;
  padding-left: 12px;
  margin-top: 10px;
  margin-bottom: 2px;
}

.client-brand-footer {
  background-color: rgba(29, 29, 26, 0.03);
  border-radius: var(--radius-md);
  margin: 12px;
  padding: 12px;
  border-top: none;
}
.client-avatar-circle {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background-color: #fff;
  border: 1px solid var(--border-color);
  color: var(--accent-color);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.75rem;
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
.badge-red {
  background-color: #ef4444;
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
  padding: 24px 32px;
  overflow-y: auto;
}

/* CLIENT MODE LAYOUTS */
.client-top-bar {
  display: flex;
  align-items: center;
  padding: 8px 0 16px 0;
  border-bottom: 1px solid var(--border-color);
  margin-bottom: 24px;
}
.icon-btn-toggle {
  background: none;
  border: none;
  font-size: 1.1rem;
  cursor: pointer;
  margin-right: 12px;
}
.client-role-badge {
  background-color: #e6f4ea;
  color: #137333;
  padding: 4px 10px;
  border-radius: 99px;
  font-size: 0.75rem;
  font-weight: 700;
  display: inline-flex;
  align-items: center;
  gap: 6px;
}
.green-dot {
  width: 6px;
  height: 6px;
  background-color: #137333;
  border-radius: 50%;
}
.top-bar-right {
  margin-left: auto;
}
.bell-notification-container {
  position: relative;
  font-size: 1.2rem;
  cursor: pointer;
}
.bell-badge {
  position: absolute;
  top: -4px;
  right: -4px;
  background-color: #ef4444;
  color: #fff;
  font-size: 0.6rem;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
}

/* Blue Notification Banner Card */
.card-blue-notif {
  background-color: #f5f3ff;
  border: 1px solid #ddd6fe;
  border-radius: var(--radius-md);
  padding: 16px;
}
.banner-title {
  font-weight: 700;
  color: #4c1d95;
  font-size: 0.9rem;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 8px;
}
.banner-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.banner-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.8rem;
  color: #5b21b6;
}
.banner-btn {
  background-color: #4c1d95;
  color: #fff;
  border: none;
  font-size: 0.7rem;
  padding: 4px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 700;
}
.banner-btn.btn-purple {
  background-color: #6d28d9;
}
.banner-btn.btn-blue-dark {
  background-color: #1e3a8a;
}

/* Stat Box Grid */
.stats-row-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-top: 12px;
}
.stat-box-card {
  background-color: #fff;
  border: 1px solid var(--border-color);
  padding: 14px;
  border-radius: var(--radius-md);
}
.stat-box-title {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: var(--text-muted);
  font-weight: 600;
}
.trend-indicator.up {
  color: #16a34a;
  font-weight: 700;
}
.stat-box-value {
  font-size: 1.4rem;
  font-weight: 800;
  margin-top: 8px;
}

/* Simulated Line Chart styles */
.simulated-chart-container {
  display: flex;
  gap: 12px;
  height: 160px;
  align-items: flex-end;
  padding: 10px 0;
}
.chart-y-axis {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  font-size: 0.7rem;
  color: var(--text-muted);
}
.chart-body-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  height: 100%;
  position: relative;
}
.chart-svg {
  width: 100%;
  height: 100px;
  margin-top: auto;
}
.chart-x-axis {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-top: 8px;
  padding: 0 10px;
}
.card-header-flex {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.chart-timeframe {
  font-size: 0.75rem;
  color: var(--text-muted);
}

/* Best Posts Table style */
.best-posts-table-wrapper {
  margin-top: 10px;
}
.best-posts-table {
  width: 100%;
  border-collapse: collapse;
}
.best-posts-table tr {
  border-bottom: 1px solid #f4f4f0;
}
.best-posts-table td {
  padding: 12px 6px;
  font-size: 0.8rem;
}
.index-num {
  font-weight: 800;
  color: var(--text-muted);
  width: 30px;
}
.post-title {
  font-weight: 600;
}
.post-stats-col {
  text-align: right;
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  color: var(--text-muted);
}

/* Weekly Schedule List */
.weekly-schedule-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 12px;
}
.schedule-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
  border-bottom: 1px solid #f4f4f0;
  padding-bottom: 8px;
}
.schedule-day {
  font-size: 0.75rem;
  color: var(--text-muted);
  font-weight: 600;
}
.schedule-details {
  display: flex;
  align-items: center;
  gap: 10px;
}
.post-icon-badge {
  width: 24px;
  height: 24px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
}
.color-carousel { background-color: #e0f2fe; color: #0369a1; }
.color-reel { background-color: #f3e8ff; color: #6b21a8; }
.schedule-text {
  flex: 1;
}
.sch-title {
  font-size: 0.8rem;
  font-weight: 700;
}
.sch-meta {
  font-size: 0.7rem;
  color: var(--text-muted);
}

/* Awaiting Approvals Box */
.awaiting-approval-box {
  margin-top: 12px;
}
.post-card-preview-header {
  margin-bottom: 8px;
}
.post-preview-title {
  font-size: 0.9rem;
  font-weight: 700;
  margin-top: 4px;
}
.post-preview-caption {
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-bottom: 12px;
}
.mini-slide-preview-frame {
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  overflow: hidden;
  margin-bottom: 12px;
}
.mini-slide-content-wrapper {
  position: relative;
}
.mini-badge-slide {
  position: absolute;
  top: 6px;
  right: 6px;
  background-color: rgba(0, 0, 0, 0.6);
  color: #fff;
  font-size: 0.6rem;
  padding: 2px 6px;
  border-radius: 99px;
}
.mini-canvas {
  height: 100px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 12px;
  text-align: center;
}
.mini-canvas h5 {
  font-size: 0.75rem;
  font-weight: 800;
  color: #9a3412;
}
.mini-canvas p {
  font-size: 0.65rem;
  color: var(--text-muted);
}

.approval-action-buttons {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

/* Timeline/Contenido Simple View */
.client-content-timeline {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.timeline-group-block {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.timeline-date-label {
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--accent-color);
  margin-bottom: 4px;
}
.timeline-post-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  cursor: pointer;
  transition: transform 0.2s;
}
.timeline-post-card:hover {
  transform: translateX(4px);
}
.post-card-left {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.post-type-tag-styled {
  font-size: 0.65rem;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 4px;
  align-self: flex-start;
}
.color-carousel-tag { background-color: #ede9fe; color: #6d28d9; }
.color-reel-tag { background-color: #e0f2fe; color: #0369a1; }
.color-story-tag { background-color: #fef3c7; color: #d97706; }
.color-post-tag { background-color: #dcfce7; color: #15803d; }

.post-title-text {
  font-size: 0.95rem;
  font-weight: 700;
}
.post-meta-sub {
  font-size: 0.75rem;
  color: var(--text-muted);
}
.post-card-right {
  display: flex;
  align-items: center;
  gap: 12px;
}
.comment-count-badge {
  font-size: 0.75rem;
  background-color: #f4f4f0;
  padding: 4px 8px;
  border-radius: var(--radius-sm);
  font-weight: 600;
}
.arrow-right-ico {
  color: var(--text-muted);
}

/* Detail Drawer Modal Overlay */
.detail-drawer-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(29, 29, 26, 0.4);
  z-index: 1000;
  display: flex;
  justify-content: flex-end;
}
.detail-drawer-body {
  width: 750px;
  background-color: #fff;
  height: 100%;
  box-shadow: -4px 0 24px rgba(0,0,0,0.15);
  display: flex;
  flex-direction: column;
  animation: slideIn 0.3s ease-out;
}
@keyframes slideIn {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}

.drawer-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 24px;
  border-bottom: 1px solid var(--border-color);
}
.drawer-header-meta h3 {
  font-size: 1.25rem;
  font-weight: 800;
  margin-top: 4px;
}
.drawer-sub {
  font-size: 0.8rem;
  color: var(--text-muted);
}
.btn-close-drawer {
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
}

.drawer-content-grid {
  display: grid;
  grid-template-columns: 1.2fr 1fr;
  gap: 20px;
  padding: 24px;
  flex: 1;
  overflow-y: auto;
}

.frame-centered {
  margin: 0 auto;
  box-shadow: var(--shadow-md);
  max-width: 320px;
}
.bg-yellow-story {
  background-color: #fffbeb;
  border-color: #fde68a;
  min-height: 250px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 24px;
}
.story-category-tag {
  font-size: 0.7rem;
  font-weight: 800;
  color: #b45309;
  text-transform: uppercase;
  margin-bottom: 10px;
}
.story-hook-text {
  font-size: 1.1rem;
  font-weight: 800;
  margin-bottom: 8px;
}
.story-desc-text {
  font-size: 0.8rem;
  color: var(--text-muted);
}

.drawer-comments-box {
  display: flex;
  flex-direction: column;
  height: 320px;
}
.drawer-comments-list {
  flex: 1;
  overflow-y: auto;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 10px;
  background-color: #fafaf9;
}
.comment-bubble-item {
  background-color: #fff;
  border: 1px solid var(--border-color);
  padding: 8px 10px;
  border-radius: var(--radius-sm);
  margin-bottom: 6px;
}
.comment-bubble-header {
  display: flex;
  justify-content: space-between;
  font-size: 0.7rem;
  margin-bottom: 4px;
}
.comment-badge {
  font-weight: 700;
  color: var(--accent-color);
}
.comment-bubble-text {
  font-size: 0.75rem;
}

/* Sessions View Layout */
.sessions-section-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 700;
  font-size: 1rem;
}
.sessions-count-badge {
  background-color: #f4f4f0;
  border: 1px solid var(--border-color);
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 99px;
}
.session-main-card {
  padding: 0;
  overflow: hidden;
}
.session-card-header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  cursor: pointer;
  background-color: #fff;
}
.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}
.camera-icon-circle {
  font-size: 1.2rem;
}
.session-title-text {
  font-size: 1.05rem;
  font-weight: 800;
}
.session-subtitle-text {
  font-size: 0.75rem;
  color: var(--text-muted);
}
.header-right {
  display: flex;
  align-items: center;
  gap: 16px;
}
.dropdown-arrow-icon {
  font-size: 0.8rem;
  color: var(--text-muted);
}

.session-card-body-details {
  border-top: 1px solid var(--border-color);
  padding: 20px;
  background-color: #fafafa;
}

.session-specs-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}
.spec-col-item {
  display: flex;
  align-items: center;
  gap: 10px;
  background-color: #fff;
  border: 1px solid var(--border-color);
  padding: 12px;
  border-radius: var(--radius-md);
}
.spec-icon {
  font-size: 1.2rem;
}
.spec-label {
  font-size: 0.65rem;
  color: var(--text-muted);
  font-weight: 700;
  letter-spacing: 0.5px;
}
.spec-value {
  font-size: 0.8rem;
  font-weight: 700;
}

.section-divider-title {
  font-size: 0.85rem;
  font-weight: 700;
  text-transform: uppercase;
  color: var(--text-muted);
  letter-spacing: 0.5px;
  border-bottom: 1px dashed var(--border-color);
  padding-bottom: 6px;
  margin-bottom: 12px;
}

.roles-cards-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.role-detail-card {
  background-color: #fff;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 16px;
}
.role-pill-header {
  margin-bottom: 6px;
}
.role-pill-badge {
  background-color: #f3f0ff;
  color: #6b21a8;
  font-size: 0.65rem;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
}
.role-desc-title {
  font-size: 0.85rem;
  font-weight: 700;
  margin-bottom: 8px;
}
.role-bullet-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.role-bullet-list li {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 0.75rem;
}
.pinterest-link-style {
  color: #b91c1c;
  text-decoration: underline;
}
.italic-tip {
  font-style: italic;
  color: var(--text-muted);
}

.session-bottom-refs-row {
  display: flex;
  gap: 16px;
}
.ref-link-card {
  flex: 1;
  background-color: #fff;
  border: 1px solid var(--border-color);
  padding: 12px;
  text-align: center;
  font-size: 0.8rem;
  font-weight: 600;
  border-radius: var(--radius-md);
  cursor: pointer;
}

/* Monthly Calendar Tab styling */
.calendar-categories-filters {
  display: flex;
  gap: 8px;
}
.filter-category-btn {
  background: #fff;
  border: 1px solid var(--border-color);
  padding: 6px 14px;
  border-radius: var(--radius-md);
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
}
.active-carrusel { border-left: 3px solid #8b5cf6; }
.active-reel { border-left: 3px solid #0284c7; }
.active-story { border-left: 3px solid #d97706; }
.active-post { border-left: 3px solid #16a34a; }

.calendar-nav-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}
.nav-arrow-btn {
  background: none;
  border: none;
  font-size: 1rem;
  cursor: pointer;
  color: var(--text-muted);
}
.month-title {
  font-size: 1.15rem;
  font-weight: 800;
}
.calendar-table-container {
  overflow-x: auto;
}
.calendar-table-grid {
  width: 100%;
  border-collapse: collapse;
}
.calendar-table-grid th {
  padding: 10px;
  font-size: 0.75rem;
  text-transform: uppercase;
  color: var(--text-muted);
  font-weight: 700;
  border-bottom: 1px solid var(--border-color);
}
.calendar-table-grid td {
  border: 1px solid var(--border-color);
  height: 80px;
  width: 14%;
  padding: 6px;
  vertical-align: top;
}
.day-number {
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--text-muted);
}
.disabled-day {
  background-color: #fafaf9;
}
.today-highlight {
  background-color: #eff6ff;
}
.circle-blue {
  background-color: #2563eb;
  color: #fff !important;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.calendar-event-tag {
  font-size: 0.65rem;
  padding: 2px 6px;
  border-radius: 4px;
  margin-top: 4px;
  font-weight: 600;
}
.tag-story { background-color: #fef3c7; color: #d97706; }
.tag-carousel { background-color: #f3e8ff; color: #6b21a8; }
.tag-reel { background-color: #e0f2fe; color: #0369a1; }
.tag-post { background-color: #dcfce7; color: #15803d; }


/* General module view styles */
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

/* AI Pillars visual result section */
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

/* Client Detail Layout (Agency view) */
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
  z-index: 10;
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

/* Animations */
.animate-fade-in {
  animation: fadeIn 0.2s ease-in-out;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* Global utility */
.mt-3 { margin-top: 12px; }
.mt-4 { margin-top: 16px; }
.ml-2 { margin-left: 8px; }
.text-center { text-align: center; }
.w-100 { width: 100%; }
.mt-auto { margin-top: auto; }
</style>
