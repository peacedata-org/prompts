# Adding This Repository as a Sub-repository Using Git Submodules

1. Navigate to the parent repository:
   ```bash
   cd /path/to/parent/repo
   ```

2. Add this repository as a submodule:
   ```bash
   git submodule add https://github.com/peacedata-org/prompts.git submodules/prompts

   ```

3. Commit the changes:
   ```bash
   git add .
   git commit -m "Add prompts as submodule"
   ```
