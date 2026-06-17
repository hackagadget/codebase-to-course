# Source Viewer Modal

When linking to external source code repositories (OpenGrok, GitHub, GitLab, etc.), use an inline modal viewer instead of navigating away from the course.

## When to Use

- "Files to Explore" sections listing source files
- Inline code references that link to full source
- Any link to an external code browser where the user should stay in context

## Required User Input

Before implementing source links, ask the user:

1. **"Where can the source files be browsed online?"** - Get the base URL
2. **"What type of source browser is it?"** - OpenGrok, GitHub, GitLab, Gitea, cgit, or other

Adjust the implementation based on their answers:

| Browser Type | URL Pattern Example |
|--------------|---------------------|
| OpenGrok | `https://opengrok.example.com/source/xref/project/` + `path/to/file.c` |
| GitHub | `https://github.com/owner/repo/blob/branch/` + `path/to/file.c` |
| GitLab | `https://gitlab.com/owner/repo/-/blob/branch/` + `path/to/file.c` |
| Gitea | `https://gitea.example.com/owner/repo/src/branch/main/` + `path/to/file.c` |
| cgit | `https://git.example.com/repo/tree/` + `path/to/file.c` |

If the user doesn't have an online source browser, omit the modal and just list files as plain text.

## HTML Structure

```html
<!-- File List -->
<div class="file-list">
    <ul>
        <li>
            <a class="file-link" data-file="path/to/file.c">path/to/file.c</a>
            <span class="file-desc">Description of file purpose</span>
        </li>
        <!-- More files... -->
    </ul>
</div>

<!-- Modal (place once, near end of content before </main>) -->
<div class="modal-overlay" id="source-modal">
    <div class="modal-content">
        <div class="modal-header">
            <span class="modal-title" id="modal-title">Loading...</span>
            <button class="modal-close" id="modal-close" aria-label="Close">&times;</button>
        </div>
        <div class="modal-body">
            <div class="modal-loader" id="modal-loader">
                <div class="spinner"></div>
                <div class="modal-loader-text">Loading source...</div>
            </div>
            <iframe id="modal-iframe" src="about:blank"></iframe>
        </div>
        <div class="modal-footer">
            <a id="modal-external" href="#" target="_blank">Open in new tab ↗</a>
        </div>
    </div>
</div>
```

## CSS

```css
/* ====== Source File Links ====== */
.file-list {
    background: var(--color-code-bg);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    padding: var(--space-lg);
    margin: var(--space-lg) 0;
}

.file-list ul {
    list-style: none;
    margin: 0;
    padding: 0;
}

.file-list li {
    padding: var(--space-sm) 0;
    border-bottom: 1px solid var(--color-border);
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: var(--space-sm);
}

.file-list li:last-child {
    border-bottom: none;
}

.file-link {
    font-family: var(--font-mono);
    font-size: 0.95rem;
    color: var(--color-info);
    text-decoration: none;
    cursor: pointer;
    transition: var(--transition-fast);
}

.file-link:hover {
    color: var(--color-accent);
    text-decoration: underline;
}

.file-desc {
    color: var(--color-text-muted);
    font-size: 0.9rem;
}

/* ====== Source Viewer Modal ====== */
.modal-overlay {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    z-index: 1000;
    justify-content: center;
    align-items: center;
    padding: var(--space-lg);
}

.modal-overlay.active {
    display: flex;
}

.modal-content {
    background: white;
    border-radius: 12px;
    width: 100%;
    max-width: 1200px;
    height: 85vh;
    display: flex;
    flex-direction: column;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: var(--space-md) var(--space-lg);
    border-bottom: 1px solid var(--color-border);
    background: var(--color-code-bg);
    border-radius: 12px 12px 0 0;
}

.modal-title {
    font-family: var(--font-mono);
    font-size: 0.95rem;
    font-weight: 500;
    color: var(--color-text);
}

.modal-close {
    background: none;
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    color: var(--color-text-muted);
    padding: var(--space-xs) var(--space-sm);
    border-radius: 4px;
    transition: var(--transition-fast);
    line-height: 1;
}

.modal-close:hover {
    background: var(--color-border);
    color: var(--color-accent);
}

.modal-body {
    flex: 1;
    overflow: hidden;
    position: relative;
}

.modal-body iframe {
    width: 100%;
    height: 100%;
    border: none;
}

.modal-loader {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: white;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: var(--space-md);
    z-index: 10;
    transition: opacity 0.3s ease;
}

.modal-loader.hidden {
    opacity: 0;
    pointer-events: none;
}

.spinner {
    width: 48px;
    height: 48px;
    border: 4px solid var(--color-border);
    border-top-color: var(--color-accent);
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

.modal-loader-text {
    color: var(--color-text-muted);
    font-size: 0.95rem;
}

.modal-footer {
    padding: var(--space-sm) var(--space-lg);
    border-top: 1px solid var(--color-border);
    background: var(--color-code-bg);
    border-radius: 0 0 12px 12px;
    text-align: right;
}

.modal-footer a {
    font-size: 0.85rem;
    color: var(--color-info);
    text-decoration: none;
}

.modal-footer a:hover {
    text-decoration: underline;
}
```

## JavaScript

```javascript
// Source Viewer Modal functionality
// CONFIGURE: Set the base URL from user input
const SOURCE_BASE_URL = 'https://example.com/source/path/'; // Replace with user-provided URL

const modal = document.getElementById('source-modal');
const modalTitle = document.getElementById('modal-title');
const modalIframe = document.getElementById('modal-iframe');
const modalExternal = document.getElementById('modal-external');
const modalClose = document.getElementById('modal-close');
const modalLoader = document.getElementById('modal-loader');

function openModal(filePath) {
    const url = SOURCE_BASE_URL + filePath;
    modalTitle.textContent = filePath;
    modalLoader.classList.remove('hidden');
    modalIframe.src = url;
    modalExternal.href = url;
    modal.classList.add('active');
    document.body.style.overflow = 'hidden';
}

function closeModal() {
    modal.classList.remove('active');
    modalIframe.src = 'about:blank';
    modalLoader.classList.remove('hidden');
    document.body.style.overflow = '';
}

// Hide loader when iframe finishes loading
modalIframe.addEventListener('load', () => {
    modalLoader.classList.add('hidden');
});

// File link click handlers
document.querySelectorAll('.file-link').forEach(link => {
    link.addEventListener('click', (e) => {
        e.preventDefault();
        openModal(link.dataset.file);
    });
});

// Close button
modalClose.addEventListener('click', closeModal);

// Click outside to close
modal.addEventListener('click', (e) => {
    if (e.target === modal) {
        closeModal();
    }
});

// Escape key to close
document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && modal.classList.contains('active')) {
        closeModal();
    }
});
```

## Accessibility Features

- Close button has `aria-label="Close"`
- Escape key closes modal
- Click outside modal closes it
- Body scroll is disabled when modal is open
- Loading state provides visual feedback

## Cross-Origin Considerations

Note: Some source browsers may block iframe embedding via `X-Frame-Options` or CSP headers. If the iframe fails to load:
1. The "Open in new tab" link still works
2. Consider using a proxy or asking the source browser admin to allow embedding
3. As fallback, display direct links without the modal
