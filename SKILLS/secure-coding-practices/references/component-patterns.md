# React + Tailwind Component Patterns

## Base Setup
All output files are single-file React components using:
- React 18 via CDN (`https://unpkg.com/react@18/umd/react.development.js`)
- ReactDOM via CDN
- Tailwind via CDN play (`https://cdn.tailwindcss.com`)
- lucide-react for icons (via unpkg)

Entry point `index.html` structure:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>[Project Name] — [Admin/User] Portal</title>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50 text-gray-900">
  <div id="root"></div>
  <script type="text/babel" src="./App.jsx"></script>
</body>
</html>
```

## RoleGate Pattern
```jsx
const ROLE_HIERARCHY = { owner: 4, super_admin: 3, admin: 2, user: 1 };

function RoleGate({ allowedRoles, currentRole, children, fallback = null }) {
  const allowed = allowedRoles.some(r => 
    ROLE_HIERARCHY[currentRole] >= ROLE_HIERARCHY[r]
  );
  return allowed ? children : fallback;
}
```

## Sidebar Pattern
```jsx
function Sidebar({ modules, currentModule, onNavigate, role }) {
  return (
    <aside className="w-64 bg-gray-900 text-white min-h-screen p-4 flex flex-col">
      <div className="mb-8">
        <h1 className="text-xl font-bold">[Project Name]</h1>
        <span className="text-xs text-gray-400 uppercase tracking-wide">{role}</span>
      </div>
      <nav className="flex-1 space-y-1">
        {modules.map(mod => (
          <button
            key={mod.id}
            onClick={() => onNavigate(mod.id)}
            className={`w-full text-left px-3 py-2 rounded-lg text-sm transition-colors ${
              currentModule === mod.id 
                ? 'bg-indigo-600 text-white' 
                : 'text-gray-300 hover:bg-gray-800'
            }`}
          >
            {mod.label}
          </button>
        ))}
      </nav>
    </aside>
  );
}
```

## Module Page Pattern
Every admin page for a module follows this layout:
```jsx
function ModulePage({ module, role }) {
  return (
    <div className="p-6 max-w-6xl mx-auto">
      {/* Module Header */}
      <div className="mb-6">
        <div className="flex items-center gap-3 mb-2">
          <h2 className="text-2xl font-bold">{module.name}</h2>
          <span className={`text-xs px-2 py-1 rounded-full font-medium ${roleColor(module.minRole)}`}>
            {module.minRole}
          </span>
        </div>
        <p className="text-gray-600">{module.description}</p>
      </div>

      {/* Dependency Badges */}
      <div className="flex flex-wrap gap-2 mb-6">
        {module.dependsOn.map(dep => (
          <span key={dep} className="bg-blue-100 text-blue-700 text-xs px-2 py-1 rounded">
            Depends on: {dep}
          </span>
        ))}
        {module.dependedOnBy.map(dep => (
          <span key={dep} className="bg-purple-100 text-purple-700 text-xs px-2 py-1 rounded">
            Used by: {dep}
          </span>
        ))}
      </div>

      {/* Feature Table */}
      <div className="bg-white rounded-xl border border-gray-200 overflow-hidden mb-6">
        <table className="w-full text-sm">
          <thead className="bg-gray-50 border-b border-gray-200">
            <tr>
              <th className="text-left px-4 py-3 font-medium text-gray-700">Feature</th>
              <th className="text-left px-4 py-3 font-medium text-gray-700">Status</th>
              <th className="text-left px-4 py-3 font-medium text-gray-700">Min Role</th>
              <th className="text-left px-4 py-3 font-medium text-gray-700">Acceptance</th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-100">
            {module.features.map(f => (
              <tr key={f.id} className="hover:bg-gray-50">
                <td className="px-4 py-3 font-medium">{f.name}</td>
                <td className="px-4 py-3">{statusBadge(f.status)}</td>
                <td className="px-4 py-3 text-gray-500 text-xs">{f.minRole}</td>
                <td className="px-4 py-3 text-gray-500 text-xs">{f.acceptance}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* Acceptance Condition Banner */}
      <div className="bg-amber-50 border border-amber-200 rounded-lg p-4 text-sm">
        <span className="font-semibold text-amber-800">Acceptance Condition: </span>
        <span className="text-amber-700">{module.acceptanceCondition}</span>
      </div>
    </div>
  );
}
```

## Feature Map (SVG-based)
Use a CSS grid for the interconnection map. Nodes are positioned by module category:
- Backend core: left column
- Integration/middleware: centre column
- Frontend/UI: right column

Color scheme:
- `bg-red-100 border-red-400` = Owner-only
- `bg-orange-100 border-orange-400` = Super Admin
- `bg-green-100 border-green-400` = Admin
- `bg-blue-100 border-blue-400` = Public/User

Draw connections with SVG `<line>` elements overlaid on the grid.

## Color Helper
```jsx
function roleColor(role) {
  const colors = {
    owner: 'bg-red-100 text-red-700',
    super_admin: 'bg-orange-100 text-orange-700',
    admin: 'bg-green-100 text-green-700',
    user: 'bg-blue-100 text-blue-700',
  };
  return colors[role] ?? 'bg-gray-100 text-gray-700';
}
```

## Status Badge
```jsx
function statusBadge(status) {
  const map = {
    complete:    'bg-green-100 text-green-700',
    in_progress: 'bg-yellow-100 text-yellow-700',
    pending:     'bg-gray-100 text-gray-500',
    blocked:     'bg-red-100 text-red-700',
  };
  return (
    <span className={`text-xs px-2 py-0.5 rounded-full font-medium ${map[status] ?? map.pending}`}>
      {status.replace('_', ' ')}
    </span>
  );
}
```
