# Example view

A complete MCP Apps view to adapt. Save it in the user's repository as a `.tsx` file.

```tsx
// views/tasks.tsx - an example @foro/app view, built into views/dist/tasks.html.
// It renders the structuredContent of the tool that mounted it, calls another
// tool of the same server to change it, and follows the host's theme and height.
import { useState } from 'react'
import { useCallTool, useLayout, useToolInfo, useViewState } from '@foro/app'

interface Task {
  id: string
  title: string
  done: boolean
}

interface TasksResult {
  tasks: Task[]
  [key: string]: unknown
}

interface TasksViewState {
  showDone: boolean
  [key: string]: unknown
}

export default function Tasks() {
  const { structuredContent, pending } = useToolInfo()
  const { theme, maxHeight, safeArea } = useLayout()
  const [viewState, setViewState] = useViewState<TasksViewState>()
  const complete = useCallTool('complete_task')
  // What complete_task returned, until the host pushes a newer tool result.
  const [latest, setLatest] = useState<{ after: unknown; result: TasksResult }>()
  const [failed, setFailed] = useState<string>()

  const dark = theme === 'dark'
  const showDone = viewState?.showDone ?? true
  const result =
    latest && latest.after === structuredContent ? latest.result : (structuredContent as TasksResult | undefined)
  const all = result?.tasks ?? []
  const tasks = all.filter((task) => showDone || !task.done)

  async function markDone(id: string) {
    setFailed(undefined)
    try {
      const next = await complete.call({ id })
      const updated = next.structuredContent as TasksResult | undefined
      if (next.isError) setFailed('the server refused the change.')
      else if (Array.isArray(updated?.tasks)) setLatest({ after: structuredContent, result: updated })
    } catch (err) {
      setFailed((err as Error).message)
    }
  }

  return (
    <div
      style={{
        boxSizing: 'border-box',
        maxHeight,
        overflowY: 'auto',
        padding: `${16 + safeArea.top}px ${16 + safeArea.right}px ${16 + safeArea.bottom}px ${16 + safeArea.left}px`,
        color: dark ? '#f5f5f5' : '#111827',
        background: dark ? '#111827' : '#ffffff',
        fontFamily: 'system-ui, sans-serif',
      }}
    >
      {pending || !result ? (
        <p>Loading tasks…</p>
      ) : all.length === 0 ? (
        <p>No tasks yet.</p>
      ) : tasks.length === 0 ? (
        <p>Every task is done.</p>
      ) : (
        <ul>
          {tasks.map((task) => (
            <li key={task.id}>
              <label>
                <input
                  type="checkbox"
                  checked={task.done}
                  disabled={task.done || complete.pending}
                  onChange={() => void markDone(task.id)}
                />
                {task.title}
              </label>
            </li>
          ))}
        </ul>
      )}
      {failed ? <p role="alert">Something went wrong: {failed}</p> : null}
      <button
        onClick={() =>
          setViewState({ showDone: !showDone }).catch((err: Error) => setFailed(err.message))
        }
      >
        {showDone ? 'Hide done' : 'Show done'}
      </button>
    </div>
  )
}
```
