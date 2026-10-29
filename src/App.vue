<template>
  <div class="layout">
    <!-- 左侧名片 -->
    <aside class="sidebar">
      <div class="tape"></div>
      <div class="profile-img">陈</div>
      <div class="name-card">
        <h1>CHEN XINYU</h1>
        <p>物联网工程 · 2024级</p>
      </div>
      <div class="info-block">
        <div style="margin-bottom:5px"><strong>身份:</strong> 物联网工程专业学生</div>
        <div style="margin-bottom:5px"><strong>生日:</strong> 2005.02</div>
        <div><strong>邮箱:</strong> <span style="cursor:pointer;text-decoration:underline" @click="copyEmail">706763331@qq.com</span></div>
      </div>
      <div class="stats-row">
        <div class="stat-item"><div class="stat-num">13</div><div class="stat-label">Skills</div></div>
        <div class="stat-item"><div class="stat-num">3</div><div class="stat-label">Projects</div></div>
        <div class="stat-item"><div class="stat-num">10</div><div class="stat-label">Awards</div></div>
      </div>
      <div class="quote-box">"用代码把想法做成能用的东西，正在探索物联网 + 大模型的交叉方向"</div>
      <div class="action-btns">
        <a href="https://github.com/0227-cxy" target="_blank" class="btn">GitHub ↗</a>
        <a href="mailto:706763331@qq.com" class="btn">Mail ✉</a>
      </div>
    </aside>

    <!-- 右侧内容 -->
    <main class="main-content">
      <!-- 关于我 -->
      <section id="about">
        <div class="section-header">
          <div class="section-title">About Me</div>
          <div class="section-subtitle">成长轨迹</div>
        </div>
        <p style="font-size:18px;margin-bottom:30px;font-weight:bold">
          我是一个喜欢"动手做出来"的人：从点亮第一块单片机，到发布第一个浏览器插件，再到给非遗平台做界面，每件作品都让我觉得代码是有温度的。
        </p>
        <div class="timeline-container">
          <div class="note-card" v-for="n in timeline" :key="n.year">
            <div class="note-year">{{ n.year }}</div>
            <div class="note-text">{{ n.text }}</div>
          </div>
        </div>
      </section>

      <!-- 技能 -->
      <section id="skills">
        <div class="section-header">
          <div class="section-title">Skills</div>
          <div class="section-subtitle">能力图谱</div>
        </div>
        <div class="skills-wrapper">
          <div class="skill-cluster" v-for="c in skillClusters" :key="c.title">
            <div class="cluster-title">{{ c.title }}</div>
            <div class="skill-item" v-for="s in c.items" :key="s.name">
              <span class="skill-name">{{ s.name }}</span>
              <div class="skill-bar-bg"><div class="skill-bar-fill" :style="{width:s.pct+'%'}"></div></div>
              <span class="skill-pct">{{ s.pct }}%</span>
            </div>
          </div>
        </div>
      </section>

      <!-- 荣誉 -->
      <section id="honors">
        <div class="section-header">
          <div class="section-title">Honors</div>
          <div class="section-subtitle">高光时刻</div>
        </div>
        <div class="honors-grid">
          <div class="ticket" :class="t.level" v-for="t in honors" :key="t.title">
            <div class="ticket-level">{{ t.levelLabel }}</div>
            <div class="ticket-title">{{ t.title }}</div>
            <div class="ticket-award">{{ t.award }}</div>
          </div>
        </div>
      </section>

      <!-- 项目 -->
      <section id="projects">
        <div class="section-header">
          <div class="section-title">Projects</div>
          <div class="section-subtitle">我的作品</div>
        </div>
        <div class="projects-collage">
          <div class="project-polaroid" v-for="p in projects" :key="p.title">
            <img :src="p.img" class="p-img" :alt="p.title">
            <span class="p-type">{{ p.type }}</span>
            <div class="p-overlay" v-html="p.overlay"></div>
            <div class="p-caption">{{ p.caption }}</div>
            <div style="text-align:center;margin-top:10px">
              <a :href="p.link" target="_blank" class="p-link">{{ p.linkText }}</a>
            </div>
          </div>
        </div>
      </section>

      <!-- 博客 -->
      <section id="blog">
        <div class="section-header">
          <div class="section-title">Blog</div>
          <div class="section-subtitle">思考碎片</div>
        </div>
        <div style="margin-bottom:20px;display:flex;flex-wrap:wrap;gap:8px">
          <button class="btn" :class="{active:blogTag==='all'}" style="padding:5px 15px;font-size:12px" @click="blogTag='all'">All</button>
          <button class="btn" :class="{active:blogTag===t}" style="padding:5px 15px;font-size:12px" v-for="t in blogTags" :key="t" @click="blogTag=t">{{ t }}</button>
        </div>
        <div class="blog-clippings">
          <div class="clipping" v-for="b in filteredBlogs" :key="b.title">
            <span class="clip-date">{{ b.date }}</span>
            <div class="clip-title">{{ b.title }}</div>
            <div class="clip-tags"><span v-for="t in b.tags" :key="t">{{ t }}</span></div>
            <div class="clip-summary">{{ b.summary }}</div>
            <div class="clip-likes">
              <button class="like-btn" :class="{liked:isLiked(b)}" @click="likeBlog(b)">
                {{ isLiked(b) ? '♥ 已赞' : '♡ 点赞' }}
              </button>
              <span>{{ likeCount(b) }} 次赞</span>
            </div>
          </div>
        </div>
      </section>

      <!-- 留言板 -->
      <section id="contact">
        <div class="guestbook-area">
          <h3 style="font-family:'Ma Shan Zheng','KaiTi','STKaiti',cursive;font-size:28px;margin-bottom:20px">Leave a Message</h3>
          <div class="gb-form">
            <input type="text" v-model="newMsg.name" placeholder="Your Name" style="width:100%;padding:10px;margin-bottom:10px;border:1px solid #ddd">
            <textarea v-model="newMsg.content" placeholder="Say something nice..."></textarea>
            <button class="btn" @click="addMessage">Post Message</button>
          </div>
          <div class="gb-list">
            <div class="gb-item" v-for="(m,i) in messages" :key="i">
              <div class="gb-author">{{ m.name }} <span style="font-weight:normal;color:#999;font-size:12px">{{ m.date }}</span></div>
              <div>{{ m.content }}</div>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// 时间线
const timeline = [
  { year: '2024.09', text: '入学物联网工程专业。系统学习 C 语言，完成程序设计课程设计；同步学习高数、大物、英语等基础课程。' },
  { year: '2025', text: '深入学习 Java、数字电路、模拟电路，夯实底层基础。' },
  { year: '2026', text: '学习 MySQL、Python、Web、单片机；用 AI 辅助开发完成浏览器插件与非遗、图书漂流两个平台项目。' }
]

// 技能
const skillClusters = [
  { title: '物联网 / 嵌入式', items: [
    { name: 'C 语言', pct: 65 }, { name: '51 单片机', pct: 60 }, { name: '传感器与基础协议', pct: 55 }
  ]},
  { title: 'Web 前端', items: [
    { name: 'HTML / CSS', pct: 78 }, { name: 'JavaScript', pct: 70 }, { name: 'Vue 3', pct: 58 }
  ]},
  { title: '后端与数据', items: [
    { name: 'Java', pct: 65 }, { name: 'Python', pct: 68 }, { name: 'MySQL', pct: 50 }, { name: 'Spring Boot', pct: 52 }
  ]},
  { title: 'AI 应用', items: [
    { name: 'AI 辅助开发 / 提示词', pct: 80 }, { name: '大模型微调（LoRA）', pct: 45 }, { name: 'AI 工具工作流', pct: 72 }
  ]}
]

// 荣誉
const honors = [
  { level: 'national', levelLabel: 'National · 2nd Prize', title: '中国好创意暨全国数字艺术设计大赛', award: '国家级二等奖' },
  { level: 'province', levelLabel: 'Province · 1st Prize', title: '全国三维数字化创新设计大赛 18 周年精英联赛（2025-2026）', award: '省级一等奖' },
  { level: 'province', levelLabel: 'Province · 2nd Prize', title: '第十六届蓝桥杯大赛', award: '省级二等奖' },
  { level: 'province', levelLabel: 'Province · 2nd Prize', title: '2025 辽宁省虚拟现实与新媒体创新设计大赛', award: '省级二等奖' },
  { level: 'province', levelLabel: 'Province · 3rd Prize', title: '2026 辽宁省三维数字化创新设计大赛', award: '省级三等奖' },
  { level: 'school', levelLabel: 'School · 1st Prize', title: '电子产品设计大赛', award: '校级一等奖' },
  { level: 'school', levelLabel: 'School · 3rd Prize', title: '挑战杯大学生创业计划竞赛', award: '校级三等奖' },
  { level: 'school', levelLabel: 'Academic', title: '优秀三好学生（2024-2025 学年）', award: '学业荣誉' },
  { level: 'cert', levelLabel: 'Certificate', title: '全国大学英语四级（CET-4）', award: '语言证书' },
  { level: 'practice', levelLabel: 'Practice', title: '2025 暑假"三下乡"社会实践', award: '社会实践' }
]

// 项目
const projects = [
  { img: '/img/cover-translate.jpg', title: '划词翻译', type: '浏览器插件',
    overlay: '<strong>Tech:</strong> JS, Manifest V3<br><strong>Feature:</strong> Auto Translate',
    caption: '划词翻译・生词本', link: 'https://github.com/0227-cxy/vocabulary-extension', linkText: '查看源码' },
  { img: '/img/cover-heritage.jpg', title: '非遗平台', type: '在线网页',
    overlay: '<strong>Tech:</strong> HTML, CSS, JS<br><strong>Theme:</strong> Culture Heritage',
    caption: '薪传坊・非遗平台', link: 'https://app-fcn9b11i9z41.miaoda.online/', linkText: '在线体验' },
  { img: '/img/cover-book.jpg', title: '图书漂流', type: '在线网页',
    overlay: '<strong>Tech:</strong> Frontend<br><strong>Goal:</strong> Book Sharing',
    caption: '图书漂流平台', link: 'https://app-fcib95axcutd.miaoda.online/', linkText: '在线体验' }
]

// 博客
const blogs = [
  { title: '从 0 搭起自己的个人网站', date: '2026-10-26', tags: ['部署', '前端'], summary: '用原生 HTML/CSS/JS 从零搭了一个清新治愈的个人网站并部署上线，记录下完整流程。', likes: 3 },
  { title: '我用 AI 做了一个浏览器插件', date: '2026-10-20', tags: ['AI', '插件'], summary: '划词翻译・生词本：从需求到 Manifest V3 插件，全程用 AI 辅助完成。', likes: 2 },
  { title: '物联网入门这一年', date: '2026-09-15', tags: ['物联网', '学习'], summary: '从 C 语言到单片机，再到用 AI 写插件和平台，整理学习路线。', likes: 1 }
]
const blogTag = ref('all')
const blogTags = ['部署', '前端', 'AI', '插件', '物联网', '学习']
const filteredBlogs = computed(() => blogTag.value === 'all' ? blogs : blogs.filter(b => b.tags.includes(blogTag.value)))

// 点赞
let siteLikes = JSON.parse(localStorage.getItem('site_likes') || '{}')
let siteLiked = JSON.parse(localStorage.getItem('site_liked') || '[]')
const likedSet = new Set(siteLiked)
const uidOf = (b) => 'p' + b.title
const isLiked = (b) => likedSet.has(uidOf(b))
const likeCount = (b) => b.likes + (siteLikes[uidOf(b)] || 0)
function likeBlog(b) {
  const uid = uidOf(b)
  if (likedSet.has(uid)) return
  likedSet.add(uid)
  siteLikes[uid] = (siteLikes[uid] || 0) + 1
  localStorage.setItem('site_likes', JSON.stringify(siteLikes))
  localStorage.setItem('site_liked', JSON.stringify([...likedSet]))
}

// 留言板
const messages = ref(JSON.parse(localStorage.getItem('site_guestbook') || '[]'))
const newMsg = ref({ name: '', content: '' })
function addMessage() {
  if (!newMsg.value.name || !newMsg.value.content) return
  messages.value.push({ ...newMsg.value, date: new Date().toLocaleDateString() })
  localStorage.setItem('site_guestbook', JSON.stringify(messages.value))
  newMsg.value = { name: '', content: '' }
}

// 复制邮箱
function copyEmail() {
  const addr = '706763331@qq.com'
  if (navigator.clipboard) navigator.clipboard.writeText(addr).then(() => alert('邮箱已复制！'))
}
</script>
