// ==UserScript==
// @name         LOFTER Helper Plus
// @name:zh-CN   LOFTER 合集/单篇导出助手（增强版）
// @namespace    https://github.com/HloYemi/LOFTER-Helper-Plus
// @version      3.1.4
// @description  LOFTER 一键导出合集/单篇。支持作者主页批量导出（基于API）、标签搜索、暂停/继续/取消、并发抓取、合集识别、关键词筛选、图片下载。
// @description:zh-CN  LOFTER 一键导出合集/单篇。支持作者主页批量导出（基于API）、标签搜索、暂停/继续/取消、并发抓取、合集识别、关键词筛选、图片下载。
// @author       Lumiarna, HloYemi
// @match        *://*.lofter.com/*
// @grant        GM_setValue
// @grant        GM_getValue
// @grant        GM.xmlHttpRequest
// @connect      api.lofter.com
// @connect      lofter.com
// @connect      lf127.net
// @connect      126.net
// @connect      127.net
// @connect      netease.com
// @connect      nos.netease.com
// @run-at       document-idle
// @license      MIT
// @require      https://cdn.jsdelivr.net/npm/fflate@0.8.2/umd/index.js
// @downloadURL https://update.greasyfork.org/scripts/597927/LOFTER%20Helper%20Plus.user.js
// @updateURL https://update.greasyfork.org/scripts/597927/LOFTER%20Helper%20Plus.meta.js
// ==/UserScript==

(function () {
  'use strict';

  const CONTENT_SELECTORS = [
    '.post-text', '.post-content', '.article-desc', '.article-content',
    '.content .text', '.text', '.ct', '.m-post',
  ];

  const API_PRODUCT = 'lofter-android-7.6.12';
  const SCROLL_MAX_ATTEMPTS = 40;
  const SCROLL_INTERVAL = 1500;
  const FETCH_DELAY = 200;
  const IMG_DELAY = 80;
  const FETCH_CONCURRENCY = 4;
  const IMG_CONCURRENCY = 4;
  const COLLECTION_PAGE_SIZE = 100;

  const isArchive = !location.pathname.includes('/post/');
  const isAuthorHome = location.pathname === '/' || location.pathname === '';

  const GROUP_STRATEGY = { MERGE: 'merge', SINGLE: 'single', SKIP: 'skip' };
  const EXPORT_MODE = { ARCHIVE: 'archive', ORIGIN: 'origin' };

  const COLLECTION_SETTINGS_KEY = 'lofter_helper_collection_settings';
  const LEGACY_COLLECTION_STRATEGY_KEY = 'lofter_helper_collection_strategy';

  // ==================== 任务控制 ====================
  const taskControl = {
    paused: false,
    cancelled: false,
    pauseResolvers: [],
    reset() { this.paused = false; this.cancelled = false; this.pauseResolvers = []; },
    pause() { this.paused = true; },
    resume() {
      this.paused = false;
      const resolvers = this.pauseResolvers;
      this.pauseResolvers = [];
      resolvers.forEach(r => r());
    },
    cancel() { this.cancelled = true; this.resume(); },
    async checkpoint() {
      if (this.cancelled) throw new Error('CANCELLED');
      while (this.paused && !this.cancelled) {
        await new Promise(r => this.pauseResolvers.push(r));
      }
      if (this.cancelled) throw new Error('CANCELLED');
    },
    isCancelled() { return this.cancelled; },
  };

  // ==================== 设置 ====================
  function readCollectionStrategies() {
    const raw = GM_getValue(COLLECTION_SETTINGS_KEY, '{}');
    if (raw && typeof raw === 'object') return raw;
    try {
      const parsed = JSON.parse(raw || '{}');
      return parsed && typeof parsed === 'object' ? parsed : {};
    } catch { return {}; }
  }

  const settings = {
    format: GM_getValue('lofter_helper_format', 'txt'),
    looseStrategy: GM_getValue('lofter_helper_loose_strategy', GROUP_STRATEGY.SINGLE),
    skipImages: GM_getValue('lofter_helper_skip_images', false),
    collectionStrategies: readCollectionStrategies(),
  };

  const delay = ms => new Promise(r => setTimeout(r, ms));
  const $$ = (sel, root = document) => Array.from(root.querySelectorAll(sel));
  let author = '';
  let archiveCollections = [];
  let collectionsLoadError = '';
  let apiPostLinks = [];

  function safeFileName(name, maxLen = 200) {
    return name.replace(/[\\/:*?"<>|]/g, '_').replace(/\s+/g, ' ').trim().slice(0, maxLen);
  }

  function escapeHtml(value) {
    return String(value).replace(/[&<>"']/g, char => ({
      '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;',
    }[char]));
  }

  function normalizeStrategy(value, fallback = GROUP_STRATEGY.MERGE) {
    if (value === GROUP_STRATEGY.SINGLE || value === GROUP_STRATEGY.SKIP) return value;
    return value === GROUP_STRATEGY.MERGE ? GROUP_STRATEGY.MERGE : fallback;
  }

  function persistCollectionStrategies() {
    GM_setValue(COLLECTION_SETTINGS_KEY, JSON.stringify(settings.collectionStrategies));
  }

  function getCollectionStrategy(collectionName) {
    return normalizeStrategy(settings.collectionStrategies[collectionName], GROUP_STRATEGY.MERGE);
  }

  function setCollectionStrategy(collectionName, strategy) {
    settings.collectionStrategies[collectionName] = normalizeStrategy(strategy, GROUP_STRATEGY.MERGE);
    persistCollectionStrategies();
  }

  function getArchiveBlogdomain() { return location.hostname; }

  function normalizeCollectionMeta(collection) {
    const name = collection?.name?.trim();
    if (!name) return null;
    return {
      id: String(collection.id ?? ''),
      name,
      postCount: Number(collection.postCount) || 0,
    };
  }

  function extractCollectionList(response) {
    if (Array.isArray(response)) return response.map(normalizeCollectionMeta).filter(Boolean);
    if (!response || typeof response !== 'object') return [];
    const candidates = [response.collections, response.postCollections, response.postCollectionList, response.data, response.list, response.result];
    for (const candidate of candidates) {
      if (Array.isArray(candidate)) return candidate.map(normalizeCollectionMeta).filter(Boolean);
    }
    return [];
  }

  function syncCollectionStrategies(collections) {
    const next = {};
    const legacyDefault = normalizeStrategy(GM_getValue(LEGACY_COLLECTION_STRATEGY_KEY, GROUP_STRATEGY.MERGE), GROUP_STRATEGY.MERGE);
    for (const collection of collections) {
      next[collection.name] = normalizeStrategy(settings.collectionStrategies[collection.name], legacyDefault);
    }
    settings.collectionStrategies = next;
    persistCollectionStrategies();
  }

  function getPageAuthor() {
    if (isArchive) {
      const links = $$('.w-bttl2.w-bttl-hd > a');
      return links[links.length - 1]?.textContent.trim();
    }
    return document.querySelector('h1 > a, .m-nick > a')?.textContent.trim();
  }

  function sequenceWidth(value) { return String(Math.max(1, value || 0)).length; }

  function imageExt(url) {
    const m = url.match(/\.(jpe?g|png|gif|webp|bmp)/i);
    return m ? m[1].toLowerCase() : 'jpg';
  }

  function rewriteImageHost(url) {
    const m = url.match(/^https?:\/\/nos\.netease\.com\/(imglf\d+|img)\/(.+)$/);
    if (m) return `https://${m[1]}.lf127.net/${m[2]}`;
    const m2 = url.match(/^https?:\/\/nos\.netease\.com\/(.+)$/);
    if (m2) return `https://lf127.net/${m2[1]}`;
    return url;
  }

  function downloadBlob(blob, fileName) {
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = fileName;
    a.click();
    setTimeout(() => URL.revokeObjectURL(a.href), 10_000);
  }

  function formatDate(ts) {
    const d = new Date(ts);
    const pad = n => String(n).padStart(2, '0');
    return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}`;
  }

  function isMd() { return settings.format === 'markdown'; }
  function fileExt() { return isMd() ? 'md' : 'txt'; }

  function formatArticle(a, { showTitle = true } = {}) {
    if (isMd()) {
      const imgBlock = a.images?.length ? a.images.map(src => `![](${src})`).join('\n') + '\n\n' : '';
      const lines = [];
      if (showTitle) lines.push(`## ${a.title}`, '');
      lines.push(`> 原文地址：${a.url}`, `> 发布时间：${a.publishTime}`, '', imgBlock + a.content.trim());
      return lines.join('\n');
    }
    const lines = [];
    if (showTitle) lines.push(`☆ ${a.title}`, '');
    lines.push(`原文地址：${a.url}`, `发布时间：${a.publishTime}`, '', a.content.trim());
    return lines.join('\n');
  }

  function buildSinglePrefix(article) {
    const parts = ['LOFTER'];
    if (author) parts.push(author);
    if (article.collectionName) parts.push(article.collectionName);
    parts.push(article.title);
    return safeFileName(parts.join('_'));
  }

  const FileNames = {
    singleTextFile(article) { return `${buildSinglePrefix(article)}.${fileExt()}`; },
    singleImageFile(article, index, url, indexWidth) {
      const imageIndexPadded = String(index + 1).padStart(indexWidth, '0');
      const articleUrl = (article.url || location.href).replace(/^https?:\/\//, '');
      const safeUrl = safeFileName(articleUrl);
      return `${buildSinglePrefix(article)}_${safeUrl}_${imageIndexPadded}.${imageExt(url)}`;
    },
    archiveFolder({ keyword = '' } = {}) {
      const raw = keyword ? `${keyword}_${author}` : author;
      return safeFileName(raw);
    },
    archiveMergedTextFile(folder, collectionName = '') {
      const raw = collectionName ? `${collectionName}_${author}` : `散章_${author}`;
      return `${folder}/${safeFileName(raw)}.${fileExt()}`;
    },
    archiveEntryPrefix(article, { itemWidth, seq }) {
      const posPadded = String(article.pos).padStart(itemWidth, '0');
      const seqPadded = String(seq).padStart(itemWidth, '0');
      return article.collectionName
        ? `${safeFileName(article.collectionName)}/${posPadded}_${safeFileName(article.title)}`
        : `${seqPadded}_${safeFileName(article.title)}`;
    },
    archiveArticleTextFile(folder, article, ctx) {
      const prefix = this.archiveEntryPrefix(article, ctx);
      return `${folder}/${(prefix)}.${fileExt()}`;
    },
    archiveImageFile(folder, article, ctx, imgIndex, url, imageWidth) {
      const prefix = this.archiveEntryPrefix(article, ctx);
      const imageIndexPadded = String(imgIndex + 1).padStart(imageWidth, '0');
      const articleUrl = (article.url || location.href).replace(/^https?:\/\//, '');
      const safeUrl = safeFileName(articleUrl);
      return `${folder}/${prefix}_${safeUrl}_${imageIndexPadded}.${imageExt(url)}`;
    },
  };

  function htmlToMarkdown(node) {
    if (node.nodeType === Node.TEXT_NODE) return node.textContent;
    if (node.nodeType !== Node.ELEMENT_NODE) return '';
    const tag = node.tagName.toLowerCase();
    const children = () => Array.from(node.childNodes).map(htmlToMarkdown).join('');
    switch (tag) {
      case 'br': return '\n';
      case 'b': case 'strong': return `**${children().trim()}**`;
      case 'i': case 'em': return `*${children().trim()}*`;
      case 'a': {
        const href = node.getAttribute('href') || '';
        const text = children().trim();
        return href ? `[${text}](${href})` : text;
      }
      case 'img': {
        const src = node.getAttribute('src') || node.getAttribute('data-src') || '';
        const alt = node.getAttribute('alt') || '';
        return src ? `![${alt}](${src})` : '';
      }
      case 'h1': case 'h2': case 'h3': case 'h4': case 'h5': case 'h6': {
        const level = '#'.repeat(Number(tag[1]));
        return `\n${level} ${children().trim()}\n`;
      }
      case 'blockquote': {
        const inner = children().trim().split('\n').map(l => `> ${l}`).join('\n');
        return `\n${inner}\n`;
      }
      case 'pre': {
        const code = node.querySelector('code');
        const text = code ? code.textContent : node.textContent;
        return `\n\`\`\`\n${text.trim()}\n\`\`\`\n`;
      }
      case 'code': return `\`${node.textContent}\``;
      case 'ul': return '\n' + Array.from(node.children).map(li => `- ${htmlToMarkdown(li).trim()}`).join('\n') + '\n';
      case 'ol': return '\n' + Array.from(node.children).map((li, i) => `${i + 1}. ${htmlToMarkdown(li).trim()}`).join('\n') + '\n';
      case 'li': return children();
      case 'p': case 'div': return `${children().trim()}\n\n`;
      default: return children();
    }
  }

  function extractText(root) {
    for (const sel of CONTENT_SELECTORS) {
      const elems = root.querySelectorAll(sel);
      if (!elems.length) continue;
      const text = Array.from(elems).map(e => isMd() ? htmlToMarkdown(e).trim() : e.textContent.trim()).join('\n\n');
      if (text.length > 0) return text;
    }
    return '';
  }

  function getCookie(name) {
    const m = document.cookie.match(new RegExp('(?:^|; )' + name.replace(/[.*+?^${}()|[\]\\]/g, '\\$&') + '=([^;]*)'));
    return m ? decodeURIComponent(m[1]) : '';
  }

  function parseLofterUrl(url) {
    const m = url.match(/https?:\/\/([^/]+\.lofter\.com)\/post\/[0-9a-f]+_([0-9a-f]+)/);
    return m ? { blogdomain: m[1], postId: parseInt(m[2], 16) } : null;
  }

  // ==================== API ====================
  async function fetchCollectionPosts(collectionId, offset = 0) {
    await taskControl.checkpoint();
    const auth = getCookie('LOFTER-PHONE-LOGIN-AUTH');
    const url = `https://api.lofter.com/v1.1/postCollection.api?product=${API_PRODUCT}`;
    const body = `method=getCollectionDetail&offset=${offset}&limit=${COLLECTION_PAGE_SIZE}&collectionid=${collectionId}&order=1`;

    const resp = await GM.xmlHttpRequest({
      method: 'POST',
      url,
      headers: {
        'Accept-Encoding': 'br,gzip',
        'content-type': 'application/x-www-form-urlencoded; charset=utf-8',
        ...(auth ? { 'lofter-phone-login-auth': auth } : {}),
      },
      data: body,
      timeout: 30_000,
    });

    if (resp.status !== 200) {
      console.warn(`获取合集 ${collectionId} 文章失败 [${resp.status}]`);
      return { items: [], total: 0 };
    }
    try {
      const data = JSON.parse(resp.responseText);
      const items = data?.response?.items || [];
      const total = data?.response?.collection?.postCount || items.length;
      return { items, total };
    } catch (e) {
      console.warn(`解析合集 ${collectionId} 响应失败:`, e);
      return { items: [], total: 0 };
    }
  }

  async function fetchAuthorPostsByApi() {
    let collections = [];
    try {
      collections = await fetchAuthorCollections();
    } catch (e) {
      console.warn('获取合集列表失败:', e);
      return [];
    }
    if (!collections.length) {
      console.warn('[LOFTER Helper] 作者无公开合集');
      return [];
    }
    console.log(`[LOFTER Helper] 找到 ${collections.length} 个合集`);

    const allPosts = [];
    for (const collection of collections) {
      await taskControl.checkpoint();
      try {
        let offset = 0;
        let fetched = 0;
        while (true) {
          const { items, total } = await fetchCollectionPosts(collection.id, offset);
          if (!items.length) break;

          for (const item of items) {
            const post = item.post || item;
            const url = post.blogPageUrl || post.postUrl || '';
            const title = post.title || post.noticeLinkTitle || '';
            const tags = Array.isArray(post.tagList) ? post.tagList.filter(Boolean).join(' ') : '';

            if (url) {
              allPosts.push({
                url,
                title,
                tags,
                postId: post.id || post.postId || 0,
                collectionName: collection.name,
              });
            }
          }

          fetched += items.length;
          if (fetched >= total || items.length < COLLECTION_PAGE_SIZE) break;
          offset += COLLECTION_PAGE_SIZE;
          await delay(500);
        }
        console.log(`[LOFTER Helper] 合集「${collection.name}」获取 ${fetched} 篇`);
        await delay(300);
      } catch (e) {
        if (e.message === 'CANCELLED') throw e;
        console.warn(`获取合集「${collection.name}」文章异常:`, e);
      }
    }

    console.log(`[LOFTER Helper] 合计获取 ${allPosts.length} 篇文章`);
    return allPosts;
  }

  async function fetchPostDetail(url) {
    await taskControl.checkpoint();
    const parsed = parseLofterUrl(url);
    if (!parsed) return null;
    const auth = getCookie('LOFTER-PHONE-LOGIN-AUTH');
    if (!auth) return null;
    const params = new URLSearchParams({ product: API_PRODUCT });
    const body = new URLSearchParams({
      supportposttypes: '1,2,3,4,5,6',
      blogdomain: parsed.blogdomain,
      postid: String(parsed.postId),
      offset: '0', requestType: '0',
      postdigestnew: '1', checkpwd: '1', needgetpoststat: '1',
    });
    try {
      const resp = await GM.xmlHttpRequest({
        method: 'POST',
        url: `https://api.lofter.com/oldapi/post/detail.api?${params}`,
        headers: {
          'User-Agent': 'LOFTER-Android 7.6.12 (V2272A; Android 13; null) WIFI',
          'lofproduct': API_PRODUCT,
          'lofter-phone-login-auth': auth,
          'Content-Type': 'application/x-www-form-urlencoded',
        },
        data: body.toString(),
        timeout: 10_000,
      });
      const data = JSON.parse(resp.responseText);
      return data.response?.posts?.[0]?.post || null;
    } catch { return null; }
  }

  async function fetchAuthorCollections() {
    const auth = getCookie('LOFTER-PHONE-LOGIN-AUTH');
    if (!auth) throw new Error('未找到 LOFTER 登录 Cookie');
    const params = new URLSearchParams({
      method: 'getCollectionList',
      needViewCount: '1',
      blogdomain: getArchiveBlogdomain(),
      product: API_PRODUCT,
    });
    const resp = await GM.xmlHttpRequest({
      method: 'GET',
      url: `https://api.lofter.com/v1.1/postCollection.api?${params.toString()}`,
      headers: {
        'Accept-Encoding': 'br,gzip',
        'content-type': 'application/x-www-form-urlencoded; charset=utf-8',
        'lofter-phone-login-auth': auth,
      },
      timeout: 10_000,
    });
    const data = JSON.parse(resp.responseText);
    return extractCollectionList(data.response);
  }

  function extractImages(post) {
    const result = [];
    const push = (u) => {
      if (!u) return;
      let clean = u.split('?')[0];
      clean = rewriteImageHost(clean);
      if (!result.includes(clean)) result.push(clean);
    };
    try {
      let links = post.photoLinks;
      if (typeof links === 'string') links = JSON.parse(links || '[]');
      if (Array.isArray(links)) {
        for (const p of links) push(p?.raw || p?.orign || p?.origin || '');
      }
    } catch (e) { console.warn('photoLinks 解析失败:', e); }
    if (result.length) return result;
    if (post.content) {
      const doc = new DOMParser().parseFromString(post.content, 'text/html');
      for (const img of doc.querySelectorAll('img')) {
        push(img.getAttribute('src') || img.getAttribute('data-src') || '');
      }
    }
    if (result.length) return result;
    try {
      let first = post.firstImageUrl;
      if (typeof first === 'string') first = JSON.parse(first || '[]');
      if (Array.isArray(first)) {
        const best = first.filter(Boolean).pop();
        if (best) push(best);
      }
    } catch {}
    return result;
  }

  function parseApiArticle(post) {
    const doc = new DOMParser().parseFromString(`<div class="post-content">${post.content}</div>`, 'text/html');
    const content = extractText(doc.body);
    const images = extractImages(post);
    const captionText = (post.caption || '').replace(/<[^>]+>/g, '').trim();
    const title = post.title
      || post.noticeLinkTitle
      || captionText
      || post.postCollection?.name
      || '未命名';
    return {
      url: post.blogPageUrl || '',
      title, content, images,
      publishTime: post.publishTime ? formatDate(post.publishTime) : '',
      collectionId: post.postCollection?.id ? String(post.postCollection.id) : '',
      collectionName: post.postCollection?.name || '',
      pos: post.pos || 0,
    };
  }

  async function autoScroll() {
    let lastCount = 0, stable = 0;
    for (let i = 0; i < SCROLL_MAX_ATTEMPTS; i++) {
      await taskControl.checkpoint();
      window.scrollTo(0, document.body.scrollHeight);
      await delay(SCROLL_INTERVAL);
      const count = $$("a[href*='/post/']").length;
      if (count === lastCount) { if (++stable >= 3) break; }
      else { stable = 0; lastCount = count; }
    }
    return lastCount;
  }

  function extractArticleLinks() {
    const map = new Map();
    for (const el of $$("a[href*='/post/']")) {
      const url = el.href;
      if (!url || map.has(url)) continue;
      const h3 = el.querySelector('h3');
      const title = h3?.textContent.trim() || el.querySelector('p')?.textContent.replace(/\s+/g, ' ').trim() || '';
      map.set(url, { url, title });
    }
    return [...map.values()];
  }

  async function fetchArticle(url) {
    const post = await fetchPostDetail(url);
    if (!post) throw new Error('详情 API 未返回文章数据');
    return parseApiArticle(post);
  }

  async function fetchAll(articles, onProgress) {
    const results = new Array(articles.length);
    let completed = 0;
    let cursor = 0;

    async function worker() {
      while (true) {
        if (taskControl.isCancelled()) return;
        const i = cursor++;
        if (i >= articles.length) return;
        await taskControl.checkpoint();
        try {
          results[i] = await fetchArticle(articles[i].url);
        } catch (e) {
          if (e.message === 'CANCELLED') return;
          console.warn(`抓取失败 [${articles[i].url}]:`, e.message);
          results[i] = null;
        }
        completed++;
        onProgress?.(completed, articles.length);
        if (FETCH_DELAY > 0) await delay(FETCH_DELAY);
      }
    }

    await Promise.all(Array.from({ length: Math.min(FETCH_CONCURRENCY, articles.length) }, () => worker()));
    if (taskControl.isCancelled()) throw new Error('CANCELLED');
    return results.filter(Boolean);
  }

  function exportSingle(article) {
    downloadBlob(new Blob([`${formatArticle(article)}\n`], { type: 'text/plain;charset=utf-8' }), FileNames.singleTextFile(article));
  }

  function mergeArticles(articles, { collectionName = '', skipSort = false } = {}) {
    const sorted = skipSort ? articles : [...articles].sort((a, b) => a.pos - b.pos);
    const body = sorted.map(a => `${formatArticle(a, { showTitle: true })}\n\n${'─'.repeat(36)}\n`).join('\n');
    return isMd() && collectionName ? `# ${collectionName}\n\n${body}` : body;
  }

  async function exportArchive(articles, keyword, onProgress) {
    const folder = FileNames.archiveFolder({ keyword });
    const files = {};
    const collectionGroups = new Map();
    const looseArticles = [];
    for (const a of articles) {
      if (a.collectionName) {
        if (!collectionGroups.has(a.collectionName)) collectionGroups.set(a.collectionName, []);
        collectionGroups.get(a.collectionName).push(a);
      } else looseArticles.push(a);
    }

    for (const [name, group] of collectionGroups) {
      const strategy = getCollectionStrategy(name);
      if (strategy === GROUP_STRATEGY.SKIP) continue;
      if (strategy === GROUP_STRATEGY.MERGE) {
        files[FileNames.archiveMergedTextFile(folder, name)] = fflate.strToU8(mergeArticles(group, { collectionName: name }));
      } else {
        const sorted = [...group].sort((a, b) => a.pos - b.pos);
        const maxPos = Math.max(...sorted.map(a => a.pos || 0), 1);
        const itemWidth = sequenceWidth(maxPos);
        for (const article of sorted) {
          const ctx = { index: 0, total: sorted.length, itemWidth, seq: 0 };
          files[FileNames.archiveArticleTextFile(folder, article, ctx)] = fflate.strToU8(formatArticle(article));
        }
      }
    }

    if (settings.looseStrategy === GROUP_STRATEGY.MERGE && looseArticles.length) {
      files[FileNames.archiveMergedTextFile(folder, keyword)] = fflate.strToU8(mergeArticles([...looseArticles].reverse(), { skipSort: true }));
    } else if (settings.looseStrategy === GROUP_STRATEGY.SINGLE) {
      const looseWidth = sequenceWidth(looseArticles.length);
      for (let i = 0; i < looseArticles.length; i++) {
        const article = looseArticles[i];
        const ctx = { index: i, total: looseArticles.length, itemWidth: looseWidth, seq: i + 1 };
        files[FileNames.archiveArticleTextFile(folder, article, ctx)] = fflate.strToU8(formatArticle(article));
      }
    }

    if (!settings.skipImages) {
      const exportedArticles = articles.filter(article => {
        if (article.collectionName) return getCollectionStrategy(article.collectionName) !== GROUP_STRATEGY.SKIP;
        return settings.looseStrategy !== GROUP_STRATEGY.SKIP;
      });
      const allSorted = [...exportedArticles].sort((a, b) => {
        if (a.collectionName !== b.collectionName) return a.collectionName.localeCompare(b.collectionName);
        return a.pos - b.pos;
      });

      for (let i = 0; i < allSorted.length; i++) {
        await taskControl.checkpoint();
        const article = allSorted[i];
        if (!article.images?.length) continue;
        const imageWidth = sequenceWidth(article.images.length);
        let itemWidth, seq;
        if (article.collectionName) {
          const group = collectionGroups.get(article.collectionName);
          itemWidth = sequenceWidth(Math.max(...group.map(a => a.pos || 0), 1));
          seq = 0;
        } else {
          itemWidth = sequenceWidth(looseArticles.length);
          seq = looseArticles.indexOf(article) + 1;
        }
        const ctx = { index: i, total: allSorted.length, itemWidth, seq };

        let imgCursor = 0;
        let imgCompleted = 0;
        const totalImgs = article.images.length;

        async function imgWorker() {
          while (true) {
            if (taskControl.isCancelled()) return;
            const j = imgCursor++;
            if (j >= totalImgs) return;
            await taskControl.checkpoint();
            const imgUrl = article.images[j];
            const blob = await fetchImageAsBlob(imgUrl);
            if (blob) {
              files[FileNames.archiveImageFile(folder, article, ctx, j, imgUrl, imageWidth)] =
                new Uint8Array(await blob.arrayBuffer());
            } else {
              console.warn(`图片下载失败（已跳过）: ${imgUrl}`);
            }
            imgCompleted++;
            onProgress?.(`文章 ${i + 1}/${allSorted.length}，图片 ${imgCompleted}/${totalImgs}`);
            if (IMG_DELAY > 0) await delay(IMG_DELAY);
          }
        }
        await Promise.all(Array.from({ length: Math.min(IMG_CONCURRENCY, totalImgs) }, () => imgWorker()));
      }
    }

    if (taskControl.isCancelled()) throw new Error('CANCELLED');
    if (!Object.keys(files).length) throw new Error('当前设置会跳过全部内容，没有可导出的文章');

    // 异步压缩，避免阻塞主线程
    onProgress?.('正在打包 ZIP…');
    await delay(50);
    const zipped = await new Promise((resolve, reject) => {
      fflate.zip(files, { level: 6 }, (err, data) => {
        if (err) reject(err);
        else resolve(data);
      });
    });
    onProgress?.('正在下载 ZIP…');
    downloadBlob(new Blob([zipped], { type: 'application/zip' }), `${folder}.zip`);
  }

  async function downloadImages(article, onProgress) {
    if (!article.images?.length) return;
    const imageWidth = sequenceWidth(article.images.length);
    let cursor = 0;
    let completed = 0;

    async function worker() {
      while (true) {
        if (taskControl.isCancelled()) return;
        const i = cursor++;
        if (i >= article.images.length) return;
        await taskControl.checkpoint();
        const blob = await fetchImageAsBlob(article.images[i]);
        if (blob) downloadBlob(blob, FileNames.singleImageFile(article, i, article.images[i], imageWidth));
        completed++;
        onProgress?.(`下载图片 ${completed}/${article.images.length}`);
        if (IMG_DELAY > 0) await delay(IMG_DELAY);
      }
    }
    await Promise.all(Array.from({ length: Math.min(IMG_CONCURRENCY, article.images.length) }, () => worker()));
  }

  async function fetchImageAsBlob(imgUrl) {
    let cleanUrl = imgUrl.split('?')[0];
    cleanUrl = rewriteImageHost(cleanUrl);
    cleanUrl = cleanUrl.replace(/^http:/, 'https:');
    try {
      const resp = await GM.xmlHttpRequest({
        method: 'GET', url: cleanUrl, responseType: 'blob', timeout: 30_000,
        headers: {
          'Referer': 'https://www.lofter.com/',
          'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
          'Accept': 'image/avif,image/webp,image/apng,image/*,*/*;q=0.8',
        },
      });
      if (resp.status !== 200) {
        console.warn(`图片下载失败 [${resp.status}]: ${cleanUrl}`);
        return null;
      }
      if (!resp.response || resp.response.size === 0) {
        console.warn(`图片响应为空: ${cleanUrl}`);
        return null;
      }
      return resp.response;
    } catch (e) {
      console.warn(`图片下载异常: ${cleanUrl}`, e);
      return null;
    }
  }

  // ==================== UI ====================
  let shadow;
  let pauseBtn, cancelBtn;
  let currentExport = false;

  function initShadowHost() {
    if (shadow) return shadow;
    const host = document.createElement('div');
    host.id = 'lofter-helper-host';
    document.body.append(host);
    shadow = host.attachShadow({ mode: 'closed' });
    shadow.innerHTML = `<style>
      button { position: fixed; right: 10px; z-index: 999999; padding: 8px 14px; border: none; border-radius: 6px; color: #fff; font: 500 13px/1.4 system-ui, sans-serif; cursor: pointer; box-shadow: 0 2px 8px rgba(0,0,0,.15); transition: opacity .2s, transform .15s; white-space: nowrap; }
      button:hover { opacity: .9; transform: translateY(-1px); }
      button:active { transform: translateY(0); }
      button:disabled { opacity: .6; cursor: not-allowed; }
      .primary { top: 40px; background: #1e90ff; }
      .success { top: 40px; background: #28a745; }
      .pause-btn { top: 80px; background: #f0ad4e; }
      .cancel-btn { top: 120px; background: #d9534f; }
      .settings-btn { position: fixed; right: 120px; top: 80px; z-index: 999999; padding: 8px 12px; border: none; border-radius: 6px; background: #555; color: #fff; font-size: 15px; line-height: 1.4; cursor: pointer; box-shadow: 0 2px 8px rgba(0,0,0,.15); transition: opacity .2s, transform .3s; }
      .settings-btn:hover { opacity: .85; transform: rotate(45deg); }
      .settings-panel { display: none; position: fixed; right: 120px; top: 124px; z-index: 999999; width: 180px; padding: 16px; border-radius: 10px; background: #fff; border: 1px solid #ddd; color: #333; font: 14px/1.6 system-ui, sans-serif; box-shadow: 0 4px 20px rgba(0,0,0,.2); overflow-y: auto; overscroll-behavior: contain; scrollbar-width: thin; max-height: calc(100vh - 100px); }
      .settings-panel.open { display: block; }
      .settings-panel h3 { margin: 0 0 12px; font-size: 15px; font-weight: 600; color: #222; text-align: center; }
      .settings-panel label { display: flex; align-items: center; gap: 6px; margin: 4px 0; cursor: pointer; font-size: 13px; }
      .settings-panel input[type="radio"] { margin: 0; accent-color: #1e90ff; }
      .settings-panel .group-title { font-size: 12px; color: #888; margin: 10px 0 4px; font-weight: 500; text-align: center; }
      .settings-panel .option-row { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); column-gap: 8px; width: 100%; margin: 0 0 8px; }
      .settings-panel .option-row-2 { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); column-gap: 16px; width: 100%; margin: 0 0 8px; }
      .settings-panel .option-row label { margin: 0; min-width: 0; justify-content: center; }
      .settings-panel .option-row-2 label { margin: 0; min-width: 0; justify-content: center; }
      .settings-panel .settings-note { margin-top: 8px; font-size: 12px; color: #b26a00; text-align: center; }
      .settings-panel .checkbox-row { display: flex; align-items: center; justify-content: center; gap: 6px; margin: 6px 0 8px; font-size: 13px; }
      .settings-panel .checkbox-row input[type="checkbox"] { margin: 0; accent-color: #1e90ff; }
      .settings-panel .collection-display { font-size: 13px; color: #555; text-align: center; margin: 2px 0 4px; word-break: break-all; }
    </style>`;
    return shadow;
  }

  function createBtn(label, modifier) {
    const root = initShadowHost();
    const btn = document.createElement('button');
    btn.className = modifier;
    btn.textContent = label;
    root.append(btn);
    return btn;
  }

  function createSettingsUI() {
    const root = initShadowHost();
    const collectionGroupsHtml = isArchive && archiveCollections.length
      ? archiveCollections.map((collection, index) => {
        const groupName = `collection-strategy-${index}`;
        const strategy = getCollectionStrategy(collection.name);
        const countText = collection.postCount > 0 ? `（${collection.postCount}）` : '';
        return `
      <div class="group-title">${escapeHtml(collection.name)}${countText}</div>
      <div class="option-row">
        <label><input type="radio" name="${groupName}" value="${GROUP_STRATEGY.MERGE}" data-collection-name="${escapeHtml(collection.name)}"${strategy === GROUP_STRATEGY.MERGE ? ' checked' : ''}> 合并</label>
        <label><input type="radio" name="${groupName}" value="${GROUP_STRATEGY.SINGLE}" data-collection-name="${escapeHtml(collection.name)}"${strategy === GROUP_STRATEGY.SINGLE ? ' checked' : ''}> 单篇</label>
        <label><input type="radio" name="${groupName}" value="${GROUP_STRATEGY.SKIP}" data-collection-name="${escapeHtml(collection.name)}"${strategy === GROUP_STRATEGY.SKIP ? ' checked' : ''}> 跳过</label>
      </div>`;
      }).join('') : '';
    const collectionStatusHtml = isArchive && collectionsLoadError
      ? `<div class="settings-note">${escapeHtml(collectionsLoadError)}</div>` : '';
    const btn = document.createElement('div');
    btn.className = 'settings-btn';
    btn.textContent = '⚙';
    root.append(btn);
    const panel = document.createElement('div');
    panel.className = 'settings-panel';
    panel.innerHTML = `
      <h3>设置</h3>
      <div class="group-title">导出格式</div>
      <div class="option-row-2">
        <label><input type="radio" name="fmt" value="markdown"${settings.format === 'markdown' ? ' checked' : ''}> Markdown</label>
        <label><input type="radio" name="fmt" value="txt"${settings.format === 'txt' ? ' checked' : ''}> TXT</label>
      </div>
      <div class="checkbox-row">
        <label><input type="checkbox" name="skip-images"${settings.skipImages ? ' checked' : ''}> 跳过图片</label>
      </div>
      ${!isArchive ? `<div class="group-title">所属合集</div><div class="collection-display">检测中…</div>` : ''}
      ${isArchive ? `
      <div class="group-title">散章</div>
      <div class="option-row">
        <label><input type="radio" name="loose-strategy" value="${GROUP_STRATEGY.MERGE}"${settings.looseStrategy === GROUP_STRATEGY.MERGE ? ' checked' : ''}> 合并</label>
        <label><input type="radio" name="loose-strategy" value="${GROUP_STRATEGY.SINGLE}"${settings.looseStrategy === GROUP_STRATEGY.SINGLE ? ' checked' : ''}> 单篇</label>
        <label><input type="radio" name="loose-strategy" value="${GROUP_STRATEGY.SKIP}"${settings.looseStrategy === GROUP_STRATEGY.SKIP ? ' checked' : ''}> 跳过</label>
      </div>
      ${collectionGroupsHtml}
      ${collectionStatusHtml}` : ''}
    `;
    root.append(panel);
    btn.addEventListener('click', async () => {
      panel.classList.toggle('open');
      if (panel.classList.contains('open') && isArchive && !archiveCollections.length) {
        try {
          archiveCollections = await fetchAuthorCollections();
          syncCollectionStrategies(archiveCollections);
          if (!archiveCollections.length) collectionsLoadError = '当前作者无公开合集。';
        } catch (e) {
          collectionsLoadError = `合集配置加载失败：${e.message}`;
        }
        btn.remove();
        panel.remove();
        createSettingsUI();
      }
    });
    panel.querySelectorAll('input[name="fmt"]').forEach(radio => {
      radio.addEventListener('change', () => { settings.format = radio.value; GM_setValue('lofter_helper_format', radio.value); });
    });
    panel.querySelectorAll('input[data-collection-name]').forEach(radio => {
      radio.addEventListener('change', () => { setCollectionStrategy(radio.dataset.collectionName, radio.value); });
    });
    panel.querySelectorAll('input[name="loose-strategy"]').forEach(radio => {
      radio.addEventListener('change', () => { settings.looseStrategy = radio.value; GM_setValue('lofter_helper_loose_strategy', radio.value); });
    });
    const skipImagesCheckbox = panel.querySelector('input[name="skip-images"]');
    if (skipImagesCheckbox) {
      skipImagesCheckbox.addEventListener('change', () => {
        settings.skipImages = skipImagesCheckbox.checked;
        GM_setValue('lofter_helper_skip_images', skipImagesCheckbox.checked);
      });
    }
  }

  function updateCollectionDisplay(collectionName) {
    if (!shadow) return;
    const el = shadow.querySelector('.collection-display');
    if (!el) return;
    el.textContent = collectionName || '未加入合集';
  }

  function createTaskControls() {
    const root = initShadowHost();
    if (pauseBtn) pauseBtn.remove();
    if (cancelBtn) cancelBtn.remove();

    pauseBtn = document.createElement('button');
    pauseBtn.className = 'pause-btn';
    pauseBtn.textContent = '⏸ 暂停';
    pauseBtn.style.display = 'none';
    pauseBtn.addEventListener('click', () => {
      if (taskControl.paused) {
        taskControl.resume();
        pauseBtn.textContent = '⏸ 暂停';
      } else {
        taskControl.pause();
        pauseBtn.textContent = '▶ 继续';
      }
    });
    root.append(pauseBtn);

    cancelBtn = document.createElement('button');
    cancelBtn.className = 'cancel-btn';
    cancelBtn.textContent = '✕ 取消';
    cancelBtn.style.display = 'none';
    cancelBtn.addEventListener('click', () => {
      if (confirm('确定要取消当前导出吗？')) {
        taskControl.cancel();
      }
    });
    root.append(cancelBtn);
  }

  function showTaskControls() {
    if (pauseBtn) pauseBtn.style.display = '';
    if (cancelBtn) cancelBtn.style.display = '';
  }

  function hideTaskControls() {
    if (pauseBtn) pauseBtn.style.display = 'none';
    if (cancelBtn) cancelBtn.style.display = 'none';
    if (pauseBtn) pauseBtn.textContent = '⏸ 暂停';
  }

  async function resolveExportContext(setStatus) {
    if (!isArchive) {
      return { keyword: '', links: [{ url: location.href, title: '' }] };
    }

    if (!apiPostLinks.length) {
      setStatus('正在通过 API 获取文章列表…');
      apiPostLinks = await fetchAuthorPostsByApi();
    }

    if (apiPostLinks.length > 0) {
      setStatus(`API 获取到 ${apiPostLinks.length} 篇文章，正在筛选…`);
      const keyword = prompt(
        `已获取到 ${apiPostLinks.length} 篇文章。\n` +
        `请输入关键词（多个关键词用 | 分隔，留空则导出全部）：\n` +
        `提示：匹配标题、标签和合集名，不区分大小写`
      );
      if (keyword === null) return null;

      const keywords = keyword
        ? keyword.split('|').map(k => k.trim().toLowerCase()).filter(Boolean)
        : [];

      let links = [...apiPostLinks];
      if (keywords.length) {
        links = links.filter(a => {
          const haystack = `${a.title} ${a.tags || ''} ${a.collectionName || ''}`.toLowerCase();
          return keywords.some(k => haystack.includes(k));
        });
      }

      if (!links.length) {
        const sample = apiPostLinks
          .slice(0, 20)
          .map((a, i) => `${i + 1}. [${a.collectionName || '无合集'}] ${a.title}\n    标签: ${a.tags || '无'}`)
          .join('\n');
        alert(`未找到匹配的文章。\n\n前 20 个标题/标签如下，请参考：\n\n${sample}`);
        return null;
      }

      return { keyword: keywords.join('-'), links };
    }

    setStatus('API 获取失败，回退到页面解析…');
    const totalCount = await autoScroll();
    setStatus(`已加载 ${totalCount} 篇，正在筛选…`);
    let links = extractArticleLinks();
    const keyword = prompt('请输入关键词（多个关键词用 | 分隔，留空则导出全部）：');
    if (keyword === null) return null;
    const keywords = keyword
      ? keyword.split('|').map(k => k.trim().toLowerCase()).filter(Boolean)
      : [];
    if (keywords.length) {
      links = links.filter(a => `${a.title}`.toLowerCase().includes(keywords[0]));
    }
    if (!links.length) { alert('未找到匹配的文章'); return null; }
    return { keyword: keywords.join('-'), links };
  }

  async function fetchArticlesForExport(links, setStatus) {
    setStatus(`正在抓取正文（共 ${links.length} 篇）…`);
    return fetchAll(links, (cur, tot) => { setStatus(`抓取正文 ${cur}/${tot}…`); });
  }

  async function executeExport(mode, articles, keyword, setStatus) {
    if (mode === EXPORT_MODE.ARCHIVE) {
      setStatus(`正在打包 ${articles.length} 篇为 ZIP…`);
      await exportArchive(articles, keyword, setStatus);
      return;
    }
    exportSingle(articles[0]);
    if (!settings.skipImages && articles[0].images.length) {
      setStatus(`下载图片：${articles[0].title}`);
      await downloadImages(articles[0], setStatus);
    }
  }

  function buildCompletionMessage(mode, articles) {
    const totalImages = articles.reduce((sum, a) => sum + (a.images?.length || 0), 0);
    const skippedMsg = settings.skipImages && totalImages ? `，${totalImages} 张图片已跳过` : '';
    if (mode === EXPORT_MODE.ORIGIN) {
      const article = articles[0];
      const imgMsg = !settings.skipImages && article.images.length ? `，${article.images.length} 张图片` : '';
      return `导出完成：${article.title}${imgMsg}${skippedMsg}`;
    }
    return `导出完成，共 ${articles.length} 篇文章${skippedMsg}`;
  }

  async function runExport({ mode, btn }) {
    if (currentExport) {
      alert('已有导出任务在进行中');
      return;
    }
    currentExport = true;
    taskControl.reset();

    const defaultLabel = btn.textContent;
    const setStatus = msg => { btn.textContent = msg; };

    try {
      btn.disabled = true;
      showTaskControls();

      if (mode === EXPORT_MODE.ARCHIVE) {
        const labels = { merge: '合并', single: '单篇', skip: '跳过' };
        const lines = [`- LOFTER 导出策略`, `\t- 散章：${labels[settings.looseStrategy]}`];
        for (const [name, s] of Object.entries(settings.collectionStrategies)) lines.push(`\t- ${name}：${labels[s] || s}`);
        console.log(lines.join('\n'));
      }

      const context = await resolveExportContext(setStatus);
      if (!context) return;

      const articles = await fetchArticlesForExport(context.links, setStatus);
      if (mode === EXPORT_MODE.ORIGIN) updateCollectionDisplay(articles[0]?.collectionName);
      await executeExport(mode, articles, context.keyword, setStatus);

      alert(buildCompletionMessage(mode, articles));
    } catch (e) {
      if (e.message === 'CANCELLED') {
        alert('导出已取消');
      } else {
        alert(`导出出错：${e.message}`);
      }
    } finally {
      btn.textContent = defaultLabel;
      btn.disabled = false;
      currentExport = false;
      hideTaskControls();
      taskControl.reset();
    }
  }

  function initPageActions(actions) {
    for (const action of actions) {
      const btn = createBtn(action.label, action.modifier);
      btn.addEventListener('click', () => runExport({ mode: action.mode, btn }));
    }
    createTaskControls();
    createSettingsUI();
  }

  async function init() {
    author = getPageAuthor() || 'LOFTER';

    if (isAuthorHome) {
      initPageActions([{ label: '导出全部', modifier: 'primary', mode: EXPORT_MODE.ARCHIVE }]);
      return;
    }

    if (isArchive) {
      initPageActions([{ label: '导出全部', modifier: 'primary', mode: EXPORT_MODE.ARCHIVE }]);
    } else {
      initPageActions([{ label: '导出本篇', modifier: 'success', mode: EXPORT_MODE.ORIGIN }]);
      fetchPostDetail(location.href)
        .then(post => updateCollectionDisplay(post?.postCollection?.name || ''))
        .catch(() => updateCollectionDisplay(''));
    }
  }

  void init();
})();
