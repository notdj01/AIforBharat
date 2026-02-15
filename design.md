# Design Document: DevFlow

## 1. System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│  Next.js 15 (App Router) + React Server Components          │
│  Tailwind CSS + Shadcn/UI + Framer Motion                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     API/Backend Layer                        │
│  Next.js API Routes / FastAPI (Python)                      │
│  LangGraph (Agentic Workflows)                              │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
┌──────────────────────────┐  ┌──────────────────────┐
│   Supabase Services      │  │  External Services   │
│  - PostgreSQL            │  │  - Piston API        │
│  - Auth (OAuth)          │  │  - GitHub API        │
│  - Vector Embeddings     │  │  - WebContainers     │
└──────────────────────────┘  └──────────────────────┘
```

### Component Architecture

The application follows a modular, component-based architecture with clear separation of concerns:

- **Presentation Layer**: React Server Components and Client Components
- **Business Logic Layer**: Custom hooks, utilities, and AI agent orchestration
- **Data Layer**: Supabase client, TanStack Query for caching
- **State Management**: Zustand stores for global client state

## 2. Folder Structure

```
devflow/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth route group
│   │   ├── login/
│   │   └── signup/
│   ├── (dashboard)/              # Protected routes
│   │   ├── dashboard/
│   │   ├── roadmap/[id]/
│   │   ├── profile/
│   │   ├── discover/
│   │   └── layout.tsx
│   ├── api/                      # API routes
│   │   ├── agent/
│   │   ├── roadmap/
│   │   ├── progress/
│   │   └── execute/
│   ├── layout.tsx
│   └── page.tsx
├── components/                   # React components
│   ├── ui/                       # Shadcn/UI components
│   ├── roadmap/
│   │   ├── RoadmapRenderer.tsx
│   │   ├── NodeCard.tsx
│   │   └── MermaidChart.tsx
│   ├── dashboard/
│   │   ├── DailyQuests.tsx
│   │   ├── ProgressCircle.tsx
│   │   └── ActivityHeatmap.tsx
│   ├── agent/
│   │   ├── DiagnosticTest.tsx
│   │   ├── HelpTierPanel.tsx
│   │   └── ChatInterface.tsx
│   └── gamification/
│       ├── XPBar.tsx
│       ├── StreakCounter.tsx
│       └── LeaderboardCard.tsx
├── lib/                          # Core utilities
│   ├── supabase/
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── queries.ts
│   ├── agent/
│   │   ├── langgraph-agent.ts
│   │   ├── diagnostic.ts
│   │   ├── roadmap-generator.ts
│   │   └── help-system.ts
│   ├── utils/
│   │   ├── cn.ts
│   │   ├── xp-calculator.ts
│   │   └── github-parser.ts
│   └── hooks/
│       ├── use-roadmap.ts
│       ├── use-progress.ts
│       └── use-agent.ts
├── store/                        # Zustand stores
│   ├── user-store.ts
│   ├── roadmap-store.ts
│   └── ui-store.ts
├── types/                        # TypeScript definitions
│   ├── database.ts
│   ├── agent.ts
│   └── roadmap.ts
├── public/                       # Static assets
└── supabase/                     # Supabase migrations
    └── migrations/
```

## 3. Database Schema (Supabase/PostgreSQL)

### Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  tech_stack TEXT[],
  total_xp INTEGER DEFAULT 0,
  current_streak INTEGER DEFAULT 0,
  longest_streak INTEGER DEFAULT 0,
  league TEXT DEFAULT 'bronze',
  skill_level TEXT DEFAULT 'novice',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Roadmaps Table
```sql
CREATE TABLE roadmaps (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  skill_level TEXT NOT NULL,
  mermaid_json JSONB NOT NULL,
  is_published BOOLEAN DEFAULT FALSE,
  clone_count INTEGER DEFAULT 0,
  completion_rate DECIMAL(5,2) DEFAULT 0,
  source_repo_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_roadmaps_user_id ON roadmaps(user_id);
CREATE INDEX idx_roadmaps_published ON roadmaps(is_published);
```

### Nodes Table
```sql
CREATE TABLE nodes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  roadmap_id UUID REFERENCES roadmaps(id) ON DELETE CASCADE,
  node_key TEXT NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  node_type TEXT NOT NULL, -- 'task', 'project', 'checkpoint'
  difficulty TEXT NOT NULL,
  xp_value INTEGER DEFAULT 10,
  content JSONB,
  prerequisites TEXT[],
  position INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_nodes_roadmap_id ON nodes(roadmap_id);
```

### Progress Table
```sql
CREATE TABLE progress (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  roadmap_id UUID REFERENCES roadmaps(id) ON DELETE CASCADE,
  node_id UUID REFERENCES nodes(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'not_started', -- 'not_started', 'in_progress', 'completed'
  help_tier_used TEXT, -- 'hint', 'pseudocode', 'syntax', null
  time_spent_seconds INTEGER DEFAULT 0,
  code_submissions JSONB,
  completed_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, node_id)
);

CREATE INDEX idx_progress_user_id ON progress(user_id);
CREATE INDEX idx_progress_roadmap_id ON progress(roadmap_id);
```

### Achievements Table
```sql
CREATE TABLE achievements (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  achievement_type TEXT NOT NULL,
  achievement_name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  unlocked_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_achievements_user_id ON achievements(user_id);
```

### Daily Quests Table
```sql
CREATE TABLE daily_quests (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  quest_date DATE NOT NULL,
  quest_1 TEXT NOT NULL,
  quest_2 TEXT NOT NULL,
  quest_3 TEXT NOT NULL,
  completed_quests INTEGER[] DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, quest_date)
);

CREATE INDEX idx_daily_quests_user_date ON daily_quests(user_id, quest_date);
```

### Activity Log Table
```sql
CREATE TABLE activity_log (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  activity_date DATE NOT NULL,
  xp_earned INTEGER DEFAULT 0,
  nodes_completed INTEGER DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, activity_date)
);

CREATE INDEX idx_activity_log_user_date ON activity_log(user_id, activity_date);
```

## 4. Core Component Strategy

### RoadmapRenderer Component

The `RoadmapRenderer` is the heart of the interactive roadmap experience. It handles:

**Architecture:**
```typescript
// components/roadmap/RoadmapRenderer.tsx
interface RoadmapRendererProps {
  roadmapId: string;
  mermaidJson: MermaidStructure;
  progress: ProgressMap;
  onNodeClick: (nodeId: string) => void;
}

// Component responsibilities:
// 1. Parse mermaid_json into Mermaid syntax
// 2. Render Mermaid chart with custom styling
// 3. Overlay interactive click handlers on nodes
// 4. Apply visual states (completed, in-progress, locked)
// 5. Handle zoom/pan interactions
```

**Implementation Strategy:**
1. Use `mermaid.js` to render the flowchart from JSON
2. Post-process the rendered SVG to add click handlers
3. Apply CSS classes based on progress state
4. Use Framer Motion for node state transitions
5. Implement optimistic updates when nodes are clicked

### MermaidChart Component
```typescript
// Handles pure Mermaid rendering
// Converts JSON structure to Mermaid syntax
// Example JSON structure:
{
  "nodes": [
    { "id": "A", "label": "Start Here", "type": "start" },
    { "id": "B", "label": "Learn Basics", "type": "task" }
  ],
  "edges": [
    { "from": "A", "to": "B" }
  ]
}
```

### NodeCard Component
```typescript
// Modal/drawer that opens when a node is clicked
// Displays:
// - Task description
// - Code editor (if applicable)
// - Help tier buttons
// - Submit button
// - Progress indicator
```

## 5. AI Agent Logic (LangGraph)

### Agent State Machine

```python
# lib/agent/langgraph-agent.py

from langgraph.graph import StateGraph, END
from typing import TypedDict, Literal

class AgentState(TypedDict):
    user_id: str
    skill_level: Literal["novice", "intermediate", "expert"]
    diagnostic_result: dict
    roadmap_request: dict
    generated_roadmap: dict
    help_request: dict
    help_response: dict
    repo_url: str
    repo_analysis: dict

# Define the agent graph
workflow = StateGraph(AgentState)

# Node functions
def run_diagnostic(state: AgentState) -> AgentState:
    """
    Present code snippet challenge
    Analyze user's solution
    Classify skill level
    """
    pass

def generate_roadmap(state: AgentState) -> AgentState:
    """
    Based on skill_level and user preferences
    Generate JSON structure for Mermaid
    Include nodes with appropriate difficulty
    """
    pass

def provide_help(state: AgentState) -> AgentState:
    """
    Determine help tier needed
    Generate hint/pseudocode/syntax based on tier
    Track help usage
    """
    pass

def analyze_repo(state: AgentState) -> AgentState:
    """
    Parse GitHub repo (package.json, README)
    Extract tech stack and dependencies
    Generate learning path for codebase
    """
    pass

def adjust_difficulty(state: AgentState) -> AgentState:
    """
    Based on help_usage and completion_time
    Adjust future node difficulty
    """
    pass

# Add nodes to graph
workflow.add_node("diagnostic", run_diagnostic)
workflow.add_node("generate_roadmap", generate_roadmap)
workflow.add_node("provide_help", provide_help)
workflow.add_node("analyze_repo", analyze_repo)
workflow.add_node("adjust_difficulty", adjust_difficulty)

# Define edges (state transitions)
workflow.set_entry_point("diagnostic")
workflow.add_edge("diagnostic", "generate_roadmap")
workflow.add_conditional_edges(
    "generate_roadmap",
    lambda state: "repo" if state.get("repo_url") else "complete",
    {
        "repo": "analyze_repo",
        "complete": END
    }
)
workflow.add_edge("analyze_repo", END)

# Compile the graph
agent = workflow.compile()
```

### Diagnostic Flow (Pseudocode)

```
1. PRESENT_CHALLENGE:
   - Select code snippet based on random difficulty
   - Include intentional bug or logic flaw
   - Display in code editor

2. ANALYZE_RESPONSE:
   - Check if bug was identified
   - Evaluate solution quality
   - Measure time taken
   
3. CLASSIFY_USER:
   IF (bug_fixed AND solution_optimal AND time < 5min):
     skill_level = "expert"
   ELSE IF (bug_fixed AND time < 10min):
     skill_level = "intermediate"
   ELSE:
     skill_level = "novice"

4. GENERATE_ROADMAP:
   - Query vector DB for relevant learning paths
   - Adjust node difficulty based on skill_level
   - Create Mermaid JSON structure
   - Return to frontend
```

### Help System Flow (Pseudocode)

```
1. DETECT_STUCK_USER:
   IF (time_on_node > 15min OR help_requested):
     TRIGGER help_system

2. DETERMINE_TIER:
   IF (first_request):
     tier = "hint"
   ELSE IF (previous_tier == "hint"):
     tier = "pseudocode"
   ELSE:
     tier = "syntax"

3. GENERATE_HELP:
   SWITCH tier:
     CASE "hint":
       RETURN conceptual_guidance
     CASE "pseudocode":
       RETURN logic_structure
     CASE "syntax":
       RETURN code_snippet

4. TRACK_USAGE:
   UPDATE progress SET help_tier_used = tier
   INCREMENT user.help_usage_count
   
5. ADJUST_FUTURE_DIFFICULTY:
   IF (help_usage_count > 3 in last 5 nodes):
     DECREASE next_node_difficulty
```

## 6. UI/UX Design Patterns

### Visual Style Guide

**Color Palette:**
- Background: `bg-zinc-950` (deep dark)
- Cards: `bg-zinc-900/50` with `backdrop-blur-xl` (glassmorphism)
- Accents: `bg-gradient-to-r from-blue-500 to-purple-600`
- Text: `text-zinc-100` (primary), `text-zinc-400` (secondary)
- Success: `text-emerald-400`
- Warning: `text-amber-400`

**Typography:**
- Headings: `font-bold tracking-tight`
- Body: `font-normal leading-relaxed`
- Code: `font-mono text-sm`

### Micro-interactions

**Card Hover:**
```tsx
<div className="transition-transform hover:scale-105 duration-200">
```

**Button Press:**
```tsx
<Button className="active:scale-95 transition-transform">
```

**Skeleton Loader:**
```tsx
<div className="animate-pulse bg-gradient-to-r from-zinc-800 via-zinc-700 to-zinc-800 bg-[length:200%_100%]">
```

### Animation Patterns

**Page Transitions:**
```tsx
<AnimatePresence mode="wait">
  <motion.div
    initial={{ opacity: 0, y: 20 }}
    animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -20 }}
    transition={{ duration: 0.3 }}
  >
```

**Roadmap Node Completion:**
```tsx
// Confetti burst on completion
confetti({
  particleCount: 100,
  spread: 70,
  origin: { y: 0.6 }
});

// Node color transition
<motion.div
  animate={{ 
    backgroundColor: ["#3b82f6", "#10b981"],
    scale: [1, 1.1, 1]
  }}
  transition={{ duration: 0.5 }}
>
```

## 7. State Management Strategy

### Zustand Stores

**User Store:**
```typescript
// store/user-store.ts
interface UserState {
  user: User | null;
  xp: number;
  streak: number;
  league: string;
  addXP: (amount: number) => void;
  updateStreak: () => void;
}
```

**Roadmap Store:**
```typescript
// store/roadmap-store.ts
interface RoadmapState {
  activeRoadmap: Roadmap | null;
  progress: ProgressMap;
  setActiveRoadmap: (roadmap: Roadmap) => void;
  updateNodeProgress: (nodeId: string, status: string) => void;
}
```

### TanStack Query Usage

```typescript
// lib/hooks/use-roadmap.ts
export function useRoadmap(roadmapId: string) {
  return useQuery({
    queryKey: ['roadmap', roadmapId],
    queryFn: () => fetchRoadmap(roadmapId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

// Optimistic updates
export function useCompleteNode() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: completeNode,
    onMutate: async (nodeId) => {
      // Optimistically update UI
      await queryClient.cancelQueries(['progress']);
      const previous = queryClient.getQueryData(['progress']);
      
      queryClient.setQueryData(['progress'], (old) => ({
        ...old,
        [nodeId]: { status: 'completed' }
      }));
      
      return { previous };
    },
    onError: (err, variables, context) => {
      // Revert on error
      queryClient.setQueryData(['progress'], context.previous);
    },
  });
}
```

## 8. API Route Structure

### Agent Endpoints

**POST /api/agent/diagnostic**
- Accepts: `{ userId: string }`
- Returns: `{ challenge: CodeChallenge, challengeId: string }`

**POST /api/agent/diagnostic/submit**
- Accepts: `{ challengeId: string, solution: string }`
- Returns: `{ skillLevel: string, feedback: string }`

**POST /api/agent/roadmap/generate**
- Accepts: `{ userId: string, topic: string, skillLevel: string }`
- Returns: `{ roadmap: Roadmap, mermaidJson: object }`

**POST /api/agent/help**
- Accepts: `{ nodeId: string, tier: string }`
- Returns: `{ helpContent: string, tier: string }`

**POST /api/agent/repo-analyze**
- Accepts: `{ repoUrl: string }`
- Returns: `{ analysis: RepoAnalysis, suggestedPath: Roadmap }`

### Progress Endpoints

**GET /api/progress/:userId**
- Returns: `{ progress: ProgressMap, stats: UserStats }`

**POST /api/progress/update**
- Accepts: `{ nodeId: string, status: string, timeSpent: number }`
- Returns: `{ success: boolean, xpEarned: number }`

### Code Execution Endpoint

**POST /api/execute**
- Accepts: `{ code: string, language: string, testCases: array }`
- Returns: `{ output: string, passed: boolean, results: array }`

## 9. Security Considerations

- Row-level security (RLS) policies on all Supabase tables
- JWT token validation on all API routes
- Rate limiting on AI agent endpoints (max 10 requests/minute)
- Sandboxed code execution with timeout limits
- Input sanitization for all user-generated content
- CORS configuration for API routes
- Environment variable protection for API keys

## 10. Deployment Architecture (AWS)

- **Frontend**: AWS Amplify or Vercel (Next.js deployment)
- **Backend API**: AWS Lambda (FastAPI) or Next.js API Routes
- **Database**: Supabase (managed PostgreSQL)
- **Code Execution**: Piston API (external) or AWS Lambda (WebContainers)
- **CDN**: CloudFront for static assets
- **Monitoring**: CloudWatch for logs and metrics

## 11. Performance Optimizations

- React Server Components for initial page loads
- Incremental Static Regeneration (ISR) for public roadmaps
- Image optimization with Next.js Image component
- Code splitting and lazy loading for heavy components
- Memoization of expensive computations
- Database query optimization with proper indexing
- Caching strategy with TanStack Query
- Debounced API calls for real-time features
