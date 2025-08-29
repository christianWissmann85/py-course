# Python Mastery Bootcamp: From Zero to L3/L4 Google Engineer
## A Modern, Type-Safe, Production-Ready Python Course

---

## 🎯 Course Mission Statement

This bootcamp transforms students into professional Python developers capable of L3/L4 Google-level engineering through:
- **Typing-First Learning**: Every line of code written by hand for muscle memory
- **Modern Tooling from Day 1**: Poetry, MyPy --strict, Ruff, pytest
- **Human-AI Collaboration Focus**: Learn to code so you can effectively partner with AI
- **CLI-Native Approach**: Everything through terminal for real-world workflows
- **Production Standards**: Write code that's ready for enterprise deployment

---

## 📚 Course Philosophy

### Core Principles
1. **Learn by Typing**: No copy-paste. Type every example, every exercise.
2. **Errors are Teachers**: Every MyPy error, every Ruff warning is a learning opportunity
3. **Test-Driven Development**: Write tests first, implementation second
4. **Real-World Focus**: Every concept tied to actual use cases
5. **AI Partnership Ready**: Understand code deeply to guide AI effectively

### Student Commitment
- **40 hours/week** for full bootcamp experience
- Part-time students adjust timeline proportionally
- Daily coding exercises (minimum 2 hours hands-on)
- Weekly projects building to monthly capstones

---

## 🗂️ Repository Structure

```
py-course/
├── PYTHON_COURSE.md          # This file - Course master document
├── CLAUDE.md                 # AI assistant instructions
├── pyproject.toml            # Poetry configuration
├── .python-version           # PyEnv version lock
│
├── course/                   # Course content (READ-ONLY for students)
│   ├── 00-foundations/       # Weeks 1-8
│   ├── 01-intermediate/      # Weeks 9-16
│   ├── 02-advanced/          # Weeks 17-24
│   ├── 03-specialization/    # Weeks 25-36
│   └── bonus/                # Additional topics
│
├── exercises/                # Student workspace (WRITE here)
│   ├── foundations/
│   ├── intermediate/
│   ├── advanced/
│   └── specialization/
│
├── projects/                 # Major projects (WRITE here)
│   ├── 01-cli-toolkit/       # Month 2 project
│   ├── 02-tui-dashboard/     # Month 4 project
│   ├── 03-api-backend/       # Month 6 project
│   └── 04-ml-pipeline/       # Month 8-9 project
│
├── assessments/              # Self-evaluation checkpoints
│   ├── foundations.md
│   ├── intermediate.md
│   ├── advanced.md
│   └── final.md
│
└── resources/                # Reference materials
    ├── cheatsheets/
    ├── troubleshooting/
    └── further-reading/
```

---

## 📈 Learning Path (36 Weeks)

### Phase 0: Environment Setup (Week 0)
- Ubuntu/WSL2 setup
- PyEnv installation and configuration
- Poetry project management
- VS Code with Python extensions
- MyPy and Ruff configuration
- Git and GitHub setup

### Phase 1: Foundations (Weeks 1-8)
**Goal**: Strong Python basics with modern practices

#### Week 1-2: Python Fundamentals
- Variables and type annotations
- Basic data types with typing
- Control flow with type safety
- Functions with full signatures
- Input/output and string formatting

#### Week 3-4: Data Structures & Algorithms
- Lists, tuples, sets with generics
- Dictionaries and TypedDict
- List comprehensions with types
- Algorithm complexity (Big O)
- Common algorithms implementation

#### Week 5-6: Functions & Modules
- Advanced function concepts
- Decorators from scratch
- Generators and iterators
- Module system and packages
- Creating pip-installable packages

#### Week 7-8: Error Handling & Testing
- Exception hierarchy
- Custom exceptions
- Logging best practices
- pytest fundamentals
- Test-driven development
- Code coverage

**Capstone**: CLI tool with full test suite

### Phase 2: Intermediate (Weeks 9-16)
**Goal**: Professional Python development skills

#### Week 9-10: Object-Oriented Programming
- Classes with dataclasses
- Inheritance and composition
- Abstract base classes
- Protocols and structural subtyping
- SOLID principles

#### Week 11-12: Functional Programming
- First-class functions
- Higher-order functions
- functools mastery
- Immutability patterns
- Monads in Python

#### Week 13-14: Concurrency & Parallelism
- Threading vs multiprocessing
- async/await fundamentals
- asyncio deep dive
- Concurrent futures
- Race conditions and locks

#### Week 15-16: Files, Network & Databases
- File I/O with context managers
- JSON, CSV, XML processing
- HTTP clients and servers
- SQL with type safety
- ORMs (SQLAlchemy)

**Capstone**: Rich/Textual TUI application

### Phase 3: Advanced (Weeks 17-24)
**Goal**: Senior developer capabilities

#### Week 17-18: Advanced Python Features
- Metaclasses
- Descriptors
- Context managers (advanced)
- Dynamic code generation
- Introspection and reflection

#### Week 19-20: Performance & Optimization
- Profiling tools
- Memory management
- Cython basics
- NumPy for performance
- Algorithmic optimization

#### Week 21-22: Design Patterns & Architecture
- Gang of Four patterns
- Python-specific patterns
- Clean architecture
- Domain-driven design
- Event-driven architecture

#### Week 23-24: DevOps & Production
- Docker containerization
- CI/CD with GitHub Actions
- Monitoring and logging
- Configuration management
- Security best practices

**Capstone**: FastAPI microservice with full DevOps

### Phase 4: Specialization (Weeks 25-36)
**Goal**: Domain expertise

#### Track A: Backend Development (Weeks 25-30)
- FastAPI mastery
- GraphQL with Strawberry
- Message queues (Redis, RabbitMQ)
- Microservices patterns
- API versioning and documentation
- Authentication and authorization

#### Track B: ML/AI Engineering (Weeks 25-30)
- NumPy and Pandas mastery
- PyTorch fundamentals
- Neural network basics
- Training pipelines
- Model deployment
- MLOps practices

#### Weeks 31-36: Integration & Mastery
- Combining backend with ML
- Production ML systems
- Real-time inference APIs
- Data pipelines
- Scalability patterns
- System design

**Final Project**: Full-stack ML application

---

## 📝 Exercise Format

Each exercise follows this structure:

```markdown
# Exercise: [Name]

## Learning Objectives
- [ ] Objective 1
- [ ] Objective 2

## Requirements
- MyPy: --strict must pass
- Ruff: All checks must pass
- Tests: 100% coverage required

## Task
[Clear description]

## Skeleton Code
[Starting point - student must type this]

## Test Cases
[pytest tests that must pass]

## Definition of Done
- [ ] All tests pass
- [ ] MyPy --strict passes
- [ ] Ruff passes
- [ ] Code is documented
```

---

## 🎯 Assessment Checkpoints

### Foundation Assessment (Week 8)
- Build a CLI application
- Full test coverage
- Type-safe throughout
- Published to TestPyPI

### Intermediate Assessment (Week 16)
- TUI application with async
- Database integration
- Concurrent operations
- Performance benchmarks

### Advanced Assessment (Week 24)
- API with 99.9% uptime goal
- Containerized deployment
- CI/CD pipeline
- Monitoring dashboard

### Final Assessment (Week 36)
- System design interview prep
- Code review exercises
- Architecture decisions
- Production troubleshooting

---

## 🔧 Development Environment

### Required Tools
```bash
# Core Python
pyenv 2.3.x
python 3.12.x
poetry 1.7.x

# Linting & Type Checking
mypy 1.8.x
ruff 0.1.x
black 23.x (optional)

# Testing
pytest 7.x
pytest-cov 4.x
pytest-asyncio 0.21.x

# Development
ipython 8.x
rich 13.x
httpx 0.25.x
```

### VS Code Extensions
- Python
- Pylance
- Ruff
- MyPy Type Checker
- Python Test Explorer
- GitLens

### Configuration Files

**pyproject.toml** (starter):
```toml
[tool.poetry]
name = "py-course"
version = "0.1.0"
description = "Python Mastery Bootcamp"

[tool.poetry.dependencies]
python = "^3.12"

[tool.poetry.group.dev.dependencies]
mypy = "^1.8"
ruff = "^0.1"
pytest = "^7.4"
pytest-cov = "^4.1"
ipython = "^8.18"
rich = "^13.7"

[tool.mypy]
strict = true
python_version = "3.12"

[tool.ruff]
line-length = 100
select = ["E", "F", "W", "C90", "I", "N", "UP", "B", "SIM"]

[tool.pytest.ini_options]
addopts = "--cov --cov-report=term-missing"
```

---

## 🚀 Getting Started

### Week 0 Checklist
- [ ] Install Ubuntu/WSL2
- [ ] Set up PyEnv
- [ ] Install Python 3.12
- [ ] Install Poetry
- [ ] Configure VS Code
- [ ] Clone this repository
- [ ] Run: `poetry install`
- [ ] Run: `poetry run mypy --version`
- [ ] Run: `poetry run ruff --version`
- [ ] Create first "Hello, Type-Safe World!"

### First Exercise
```python
# exercises/foundations/hello_typed.py
def greet(name: str) -> str:
    """Type-safe greeting function."""
    return f"Hello, {name}!"

def main() -> None:
    """Entry point with proper typing."""
    user_name: str = input("Enter your name: ")
    greeting: str = greet(user_name)
    print(greeting)

if __name__ == "__main__":
    main()
```

Run checks:
```bash
poetry run mypy exercises/foundations/hello_typed.py
poetry run ruff check exercises/foundations/hello_typed.py
poetry run python exercises/foundations/hello_typed.py
```

---

## 📖 Course Style Guide

### Practical vs Academic Balance
We adopt a **"Practical-First with Academic Rigor"** approach:
- Start with real-world use case
- Explain theory when it matters
- Academic depth on demand (bonus sections)
- Always return to practical application

### Code Examples
- Every example is production-ready
- No "toy" code without real purpose
- Comments explain "why" not "what"
- Type annotations on everything

### Learning Progression
1. **See It**: Read the concept
2. **Type It**: Manually type all code
3. **Break It**: Intentionally cause errors
4. **Fix It**: Resolve with understanding
5. **Test It**: Write comprehensive tests
6. **Ship It**: Deploy or package

---

## 🎓 Graduation Criteria

### Technical Skills
- [ ] Can implement any algorithm with proper types
- [ ] Can design and build REST APIs
- [ ] Can create async applications
- [ ] Can optimize performance bottlenecks
- [ ] Can debug production issues

### Engineering Skills
- [ ] Can review code effectively
- [ ] Can design system architecture
- [ ] Can estimate and plan projects
- [ ] Can mentor junior developers
- [ ] Can collaborate with AI effectively

### Portfolio Projects
- [ ] CLI tool on PyPI
- [ ] TUI application
- [ ] Production API
- [ ] ML pipeline
- [ ] Open source contributions

---

## 🤝 Human-AI Collaboration Notes

This course prepares you for the future of development where:
- You understand code deeply enough to guide AI
- You can review AI-generated code critically
- You can specify requirements precisely
- You can debug AI mistakes
- You can architect systems AI implements

Remember: **The goal isn't to type faster than AI, but to think better than AI.**

---

## 📅 Daily Routine

### Recommended Schedule (Full-Time)
- **9:00-10:00**: Theory and reading
- **10:00-12:00**: Typing exercises
- **12:00-13:00**: Break
- **13:00-15:00**: Project work
- **15:00-16:00**: Testing and debugging
- **16:00-17:00**: Code review and reflection

### Part-Time Adaptation
- **1 hour**: Theory and exercises
- **1 hour**: Project work
- **30 min**: Testing and review

---

## 🎯 Next Steps

1. Review this document completely
2. Set up your environment (Week 0)
3. Begin with `course/00-foundations/week-01/`
4. Type every example in `exercises/`
5. Ask Claude Code to verify your work
6. Track progress in assessments

---

## 📝 Notes for Course Development

### For Task Agents
When creating course content:
- Each file should be self-contained
- Include learning objectives
- Provide typed examples
- Create exercises with solutions
- Add "Try This" challenges
- Include common mistakes section

### Content Creation Template
```markdown
# [Topic Name]

## Learning Objectives
- Understand...
- Implement...
- Apply...

## Key Concepts
[Theory explanation]

## Code Example
[Fully typed example]

## Exercise
[Student task]

## Common Pitfalls
[What to avoid]

## Further Reading
[Links and resources]
```

---

*Last Updated: [Date]*
*Version: 1.0.0*
*Student #000001: Chris*