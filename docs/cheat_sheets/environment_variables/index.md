# Environment Variables

## Python

Force Python to ignore the user site packages. On Linux, this is `~/.local/lib/python<vesion>`. This forces environments to be isolated, avoiding "cross contamination" between different installs. This is ideal when using virtual environments or containerized workflows.

```bash 
export PYTHONNOUSERSITE=1
```