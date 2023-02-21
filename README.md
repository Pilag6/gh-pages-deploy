# gh-pages-deploy

### Requirements

- Node 
- npm
- Vite

### Commands

```bash
  npm create vite@latest
  Ok to proceed? (y) y
√ Project name: ... my-app
√ Select a framework: » React
√ Select a variant: » JavaScript

```

```bash
  cd my-app
  npm install
  npm run dev 
```

- Cancel local-host with `ctrl+c`

```bash
  git init
  git add .
  git commit -m "init project with Vite"
```

### Create a new GitHub repository

Go to https://github.com/new and create a new repository.

❗️ Make sure **Public** is selected if you don't have a premium account. Otherwise, you won't be able to host your app using GitHub pages.
![Screen Shot 2022-05-15 at 16 19 34](https://user-images.githubusercontent.com/58401630/168477505-b3f2fc8f-a248-499c-9645-0c0d2bd5de35.png)

Once the repo is created, copy and paste the instructions similar to these to your terminal

```bash
  git remote add origin https://github.com/<USERNAME>/<REPO NAME>.git
  git branch -M main
  git push -u origin main
```

### Open a project with VS Code

```bash
  code .
```

### Create deployment workflow
![image](https://user-images.githubusercontent.com/79191808/220339726-239973e4-8d63-42db-a0b4-e6c9b83bddc5.png)

-New file => `.github/workflows/deploy.yml` and paste the following code:

```yml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v2

      - name: Setup Node
        uses: actions/setup-node@v1
        with:
          node-version: 16

      - name: Install dependencies
        uses: bahmutov/npm-install@v1

      - name: Build project
        run: npm run build

      - name: Upload production-ready build files
        uses: actions/upload-artifact@v2
        with:
          name: production-files
          path: ./dist

  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v2
        with:
          name: production-files
          path: ./dist

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

### Add, Commit & Push the changes in the workflow

```bash
git add .
git commit -m "add deploy workflow"
git push
```

When you go, to [Actions](https://github.com/Pilag6/gh-pages-deploy/actions) and click on the recent workflow, 
you should see that it failed, because of missing permissions:

![Screen Shot 2022-05-15 at 16 33 13](https://user-images.githubusercontent.com/58401630/168478218-93f9fda7-91ff-49fb-b96c-8aa5e682ef70.png)

### Ensure Actions have `write` permission

To fix that, go to [Actions Settings](https://github.com/Pilag6/gh-pages-deploy/settings/actions), 
select **Read and write permissions** and hit **Save**:

<img width="844" alt="Screen Shot 2022-05-15 at 16 35 37" src="https://user-images.githubusercontent.com/58401630/168478314-c11c7c49-eeeb-411c-8351-9dd2b7423681.png">

Basically, our action is going to modify the repo, so it needs the _write_ permission.

Go back to [Actions](https://github.com/Pilag6/gh-pages-deploy/actions), click on failed workflow 
and in the top-right corner click on **Re-run failed jobs**

![Screen Shot 2022-05-15 at 16 41 29](https://user-images.githubusercontent.com/58401630/168478612-3a129490-4191-4380-9eec-f51f31b77720.png)

After job run, you should be able to see a new branch `gh-pages` created in your repository.

![image](https://user-images.githubusercontent.com/79191808/220342628-a428f7d6-e04c-43ac-a24d-ee21147afe1d.png)

### Enable GitHub pages

To host the app, go to [Pages Settings](https://github.com/Pilag6/gh-pages-deploy/settings/pages), set **Source** to `gh-pages`, and hit **Save**.

![image](https://user-images.githubusercontent.com/79191808/220342844-38fac340-5cd5-4be9-924a-b41a90c4337e.png)

After a while your app should be deployed and be available at the link displayed in Pages Settings. If you want to follow the deployment process,
go to [Actions](https://github.com/Pilag6/gh-pages-deploy/actions) and **pages-build-deployment** workflow:

![image](https://user-images.githubusercontent.com/79191808/220343078-419ce4f0-413b-46a3-9cbb-51ab4f0c5efc.png)

Once deployment is done, visit the app at: `https://<YOUR_GITHUB_USER>.github.io/REPO_NAME`

### Fix assets links

You will see that something is not right, because instead of there is a blank screen. When you inspect it, you will see that some files were not found.

![Screen Shot 2022-05-15 at 17 11 42](https://user-images.githubusercontent.com/58401630/168479964-7a0c6b8e-3be5-4468-af24-a09e13d800b1.png)

This is happening, because of the subdirectory-like URL structure GitHub uses for Project Pages. Asset links are referencing the files 
in the domain root, whereas our project is located in `<ROOT>/gh-pages-deploy`. 

Also make sure that you IMGs are in src/assets folder, not in public folder

This is how the links should look like:

```
❌ Bad
https://Pilag6.github.io/assets/favicon.17e50649.svg

✅ Good
https://Pilag6.github.io/gh-pages-deploy/assets/favicon.17e50649.svg
```

Fortunately, there is a very easy fix for this. Add the following line in `vite.config.js`:

```diff
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
   plugins: [react()],
+  base: '/<REPO NAME>/'
})
```

Now, asset links will have a correct path, so commit the changes, push the code, wait for the deploy to finish and see it for yourself!

```bash
git add .
git commit -m "viteconfig base added and move imgs to src/assets folder"
git push
```

### Final 

![image](https://user-images.githubusercontent.com/79191808/220343602-5b5624b1-7243-40a7-933e-cd329691b141.png)

## Resources


- [Vite](https://vitejs.dev/)
- [GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#creating-your-site)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Video how to do it step by step](https://www.youtube.com/watch?v=MKw-IriprJY&t=253s)




