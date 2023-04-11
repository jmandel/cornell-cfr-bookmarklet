# cornell-cfr-bookmarklet
Add nav buttons to cornell cfr

```
javascript:(function() {
  class TreeNode {
    constructor(element) {
      this.element = element;
      this.id = element.id;
      this.children = [];
      this.parent = null;
      this.next = null;
      this.prev = null;
    }
  }

  function buildNodeMap() {
    const nodes = Array.from(document.querySelectorAll('.enumxml')).map((el) => new TreeNode(el));
    return new Map(nodes.map((node) => [node.id, node]));
  }

  function findAncestor(nodeMap, parentId) {
    while (parentId) {
      const ancestor = nodeMap.get(parentId);
      if (ancestor) return ancestor;
      parentId = parentId.split('_').slice(0, -1).join('_');
    }
    return null;
  }

  function parseHierarchy(nodeMap) {
    for (const node of nodeMap.values()) {
      const parentId = node.id.split('_').slice(0, -1).join('_');
      const parent = findAncestor(nodeMap, parentId);
      if (parent) {
        node.parent = parent;
        parent.children.push(node);
      }
    }
  }

  function updateSiblings(nodeMap) {
    for (const node of nodeMap.values()) {
      const siblings = node.parent ? node.parent.children : Array.from(nodeMap.values()).filter((n) => n.parent === null);
      const index = siblings.indexOf(node);
      node.prev = index > 0 ? siblings[index - 1] : null;
      node.next = index < siblings.length - 1 ? siblings[index + 1] : null;
    }
  }

  function createNavButton(target, indentLevel, onClick) {
    const button = document.createElement('button');
    button.innerText = target.id;
    button.style.marginLeft = `${indentLevel * 20}px`;
    button.onclick = onClick;
    return button;
  }

  function createNavigationMenu(node) {
    const menu = document.createElement('div');
    menu.className = 'cfr-navigation-menu';
    menu.style.cssText = 'position:absolute;background:#fff;border:1px solid #ccc;padding:10px;display:none';

    let currentNode = node;
    let level = 0;
    while (currentNode.parent) {
      let target = currentNode.parent;
      menu.appendChild(createNavButton(target, level, () => {
        target.element.scrollIntoView({ block: 'center', behavior: 'smooth' });
        target.element.style.animation = 'highlight 5s';
      }));
      currentNode = currentNode.parent;
      level++;
    }

    [node.prev, node.next].forEach((sibling, i) => {
      if (sibling) {
        let target = sibling;
        menu.appendChild(createNavButton(target, level, () => {
          target.element.scrollIntoView({ block: 'center', behavior: 'smooth' });
          target.element.style.animation = 'highlight 5s';
        }));
      }
    });

    return menu;
  }

  function enhanceNode(node) {
    const menu = createNavigationMenu(node);
    node.element.appendChild(menu);
    node.element.addEventListener('click', () => {
      menu.style.display = menu.style.display === 'none' ? 'block' : 'none';
    });
  }

  function addHighlightAnimationStyle() {
    const style = document.createElement('style');
    style.innerHTML = '@keyframes highlight { 0% { background-color: yellow; } 100% { background-color: inherit; } }\n.cfr-navigation-menu {display: flex; flex-direction: row;}\n.cfr-navigation-menu button {display: block;}';
    document.head.appendChild(style);
  }

  const nodeMap = buildNodeMap();
  parseHierarchy(nodeMap);
  updateSiblings(nodeMap);
  addHighlightAnimationStyle();
  nodeMap.forEach(enhanceNode);
})();
```
