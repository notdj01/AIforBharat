# Requirements Document: DevFlow

## Project Overview

DevFlow is an AI-powered problem-based learning platform for developers that emphasizes "learning by doing" through interactive, AI-generated roadmaps. The platform combines the polish of Linear, the gamification of Duolingo, and the utility of Codecademy with a custom AI mentor agent.

## Target Audience

- Novice to expert developers seeking structured, personalized learning paths
- Developers wanting to understand existing codebases through guided exploration
- Self-learners looking to escape "tutorial hell" with hands-on projects
- Developers interested in building in public and sharing their learning journey

## Functional Requirements

### 1. User Authentication & Profile Management

- **FR-1.1**: Support OAuth authentication via GitHub and Google using Supabase Auth
- **FR-1.2**: Display user profile with tech stack badges, total XP, and activity heatmap
- **FR-1.3**: Track and visualize user learning streaks (consecutive days)
- **FR-1.4**: Maintain user progress across multiple roadmaps and skills

### 2. AI Mentor Agent Capabilities

- **FR-2.1**: Conduct pre-flight diagnostic assessment using code snippet challenges
- **FR-2.2**: Classify users into skill levels (Novice/Intermediate/Expert) based on diagnostic results
- **FR-2.3**: Generate personalized learning roadmaps as interactive Mermaid.js flowcharts
- **FR-2.4**: Provide three-tier help system (Hint → Pseudocode → Syntax) for stuck users
- **FR-2.5**: Track help usage patterns to adjust future difficulty levels
- **FR-2.6**: Parse GitHub repositories (package.json, README) to generate codebase-specific learning paths
- **FR-2.7**: Generate DEV_LOG.md templates upon project completion with user-specific insights

### 3. Dynamic Roadmap System

- **FR-3.1**: Render roadmaps as interactive Mermaid.js flowcharts from JSON structures
- **FR-3.2**: Enable node interaction to open specific task/project views
- **FR-3.3**: Track completion status for each node in the roadmap
- **FR-3.4**: Support multiple concurrent roadmaps per user
- **FR-3.5**: Allow roadmap cloning from community library

### 4. Dashboard & Progress Tracking

- **FR-4.1**: Display "Jump Back In" quick access to most recent roadmap
- **FR-4.2**: Generate 3 daily quests (auto-generated mini-tasks)
- **FR-4.3**: Show circular progress bars for active skills
- **FR-4.4**: Visualize overall learning progress with GitHub-style activity heatmap

### 5. Gamification System

- **FR-5.1**: Award XP for completed activities (Nodes: 10 XP, Projects: 100 XP)
- **FR-5.2**: Maintain weekly leaderboards based on XP earned
- **FR-5.3**: Implement league system for competitive learning
- **FR-5.4**: Track and display achievement badges
- **FR-5.5**: Celebrate milestones with visual rewards (confetti animations)

### 6. Code Execution Environment

- **FR-6.1**: Integrate code execution via Piston API or WebContainers
- **FR-6.2**: Support multiple programming languages for code challenges
- **FR-6.3**: Provide real-time feedback on code submissions
- **FR-6.4**: Sandbox execution environment for security

### 7. Community Features

- **FR-7.1**: Enable users to publish AI-generated roadmaps to community library
- **FR-7.2**: Provide discovery page for browsing community roadmaps
- **FR-7.3**: Support roadmap cloning and customization
- **FR-7.4**: Display roadmap popularity metrics (clones, completions)

## Non-Functional Requirements

### 1. Performance

- **NFR-1.1**: Initial page load time < 2 seconds
- **NFR-1.2**: AI roadmap generation < 10 seconds
- **NFR-1.3**: Optimistic UI updates with instant feedback
- **NFR-1.4**: Skeleton loaders during AI content generation (no static spinners)

### 2. User Experience

- **NFR-2.1**: Deep dark mode as default theme with glassmorphism accents
- **NFR-2.2**: Smooth animations using Framer Motion for all transitions
- **NFR-2.3**: Micro-interactions on all interactive elements (hover effects, scale transforms)
- **NFR-2.4**: Responsive design supporting mobile, tablet, and desktop viewports
- **NFR-2.5**: Accessibility compliance (WCAG 2.1 AA standards)

### 3. Scalability

- **NFR-3.1**: Support concurrent users with minimal latency
- **NFR-3.2**: Efficient vector embedding storage and retrieval in Supabase
- **NFR-3.3**: Stateful agent workflows using LangGraph for complex interactions

### 4. Security

- **NFR-4.1**: Secure authentication flow with JWT tokens
- **NFR-4.2**: Sandboxed code execution environment
- **NFR-4.3**: Input validation and sanitization for all user inputs
- **NFR-4.4**: Rate limiting on AI agent requests

### 5. Maintainability

- **NFR-5.1**: TypeScript strict mode for type safety
- **NFR-5.2**: Component-based architecture with clear separation of concerns
- **NFR-5.3**: Comprehensive error handling and logging
- **NFR-5.4**: Modular AI agent architecture for easy capability extensions

## Technical Requirements

### Frontend Stack

- **TR-1.1**: Next.js 15 with App Router and React Server Components
- **TR-1.2**: TypeScript for type-safe development
- **TR-1.3**: Tailwind CSS for styling with Shadcn/UI component library
- **TR-1.4**: Framer Motion for animations
- **TR-1.5**: Lucide React for iconography
- **TR-1.6**: Mermaid.js for roadmap visualization
- **TR-1.7**: Canvas-Confetti for reward animations
- **TR-1.8**: Zustand for client-side state management
- **TR-1.9**: TanStack Query for server state management

### Backend Stack

- **TR-2.1**: FastAPI (Python) or Next.js API Routes for backend services
- **TR-2.2**: LangGraph for stateful agentic workflows
- **TR-2.3**: Supabase for PostgreSQL database, authentication, and vector embeddings
- **TR-2.4**: Piston API or WebContainers for code execution

### Database Requirements

- **TR-3.1**: PostgreSQL tables for users, roadmaps, nodes, progress, and achievements
- **TR-3.2**: Vector embeddings support for AI-powered content matching
- **TR-3.3**: Real-time subscriptions for live updates
- **TR-3.4**: Row-level security policies for data protection

## Data Requirements

### User Data

- User profile (name, email, avatar, tech stack preferences)
- Authentication credentials (OAuth tokens)
- XP points, streak count, league ranking
- Activity history and heatmap data

### Learning Content

- Roadmap definitions (JSON structures for Mermaid rendering)
- Node metadata (title, description, difficulty, XP value)
- Task/project specifications
- Help tier content (hints, pseudocode, syntax examples)

### Progress Tracking

- Node completion status
- Help usage statistics
- Time spent on tasks
- Code submission history
- Achievement unlocks

### Community Content

- Published roadmaps with metadata
- Clone counts and completion rates
- User ratings and feedback

## Constraints

- Must use specified tech stack without substitutions
- AI agent must respond within 10 seconds for roadmap generation
- Must support GitHub repository parsing for repo-based learning
- Must maintain user privacy and data security standards
- Must be deployable on AWS infrastructure for hackathon requirements

## Success Criteria

- Users can complete diagnostic assessment and receive personalized roadmap within 2 minutes
- AI agent provides contextually relevant help at appropriate difficulty levels
- 90% of users report improved learning outcomes compared to traditional tutorials
- Platform maintains sub-2-second page load times under normal load
- Community library grows organically with user-published roadmaps
- Users maintain learning streaks averaging 7+ days
