# Dependencies & Documentation

## Verify Before Installing

AI models have knowledge cutoffs. **Always verify library versions and APIs before use.**

- Use **architect** agent for dependency decisions
- Specify explicit versions after verifying latest stable
- Check changelogs for breaking changes on major versions

```bash
# ✅ GOOD: Explicit version after verification
npm install react@19.0.0
pip install fastapi>=0.115.0

# ❌ BAD: Without checking current version
npm install some-package
```
