# Quick-Start Guide

원문: [Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)



쉽게 사용하기 위해 [Gem-based theme](http://jekyllrb.com/docs/themes/) 으로 개발

Remote theme 기반의 GitHub 와 100% 호환

**If you enjoy this theme, please consider [supporting me](https://www.paypal.me/mmistakes) for developing and maintaining it.**



## 테마 설치 [Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#installing-the-theme)

Jekyll v3.7+ 을 운용하고, 스스로 hosting 하려면 Ruby gem 으로 테마 설치가능

**ProTip:** Minimal Mistakes 를 fork했다면, 테스트용 페이지들인 `/docs` and `/test` 제거할 것

**Note:** 테마가 [jekyll-include-cache](https://github.com/benbalter/jekyll-include-cache) 플러그인 사용하므로 `Gemfile` 에 설치되고,  array of `_config.yml`의 `plugins`에 추가되어야 함. 그렇지 않으면 `Unknown tag 'include_cached'` 오류 발생.

### Gem 기반 방법[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#gem-based-method)

With Gem-based themes, directories such as the `assets`, `_layouts`, `_includes`, and `_sass` are stored in the theme’s gem, hidden from your immediate view. This allows for easier installation and updating as you don’t have to manage any of the theme files.

To install as a Gem-based theme:

1. Add the following to your `Gemfile`:

   ```
   gem "minimal-mistakes-jekyll"
   ```

2. Fetch and update bundled gems by running the following [Bundler](https://bundler.io/) command:

   ```
   bundle
   ```

3. Set the `theme` in your project’s Jekyll `_config.yml` file:

   ```
   theme: minimal-mistakes-jekyll
   ```

To update the theme run `bundle update`.

### 원격 테마 방법[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remote-theme-method)

원격 테마는 Gem기반 테마와 비슷.

(+) `Gemfile` 필요 없음

(+) GitHub 페이지로 호스팅하는 사이트들에 최적



minimal mistakes fork하기:

**Looking for an example?** Use the [Minimal Mistakes remote theme starter](https://github.com/mmistakes/mm-github-pages-starter/generate) for the quickest method of getting a GitHub Pages hosted site up and running. Generate a new repository from the starter, replace sample content with your own, and configure as needed.

위의 [Minimal Mistakes remote theme starter](https://github.com/mmistakes/mm-github-pages-starter/generate) 을 누르면 아래와 같은 사이트로 이동. 여기서 아래처럼 repo name 을 설정하면, minimal mistakes를 fork함.

![image-20200720231237796](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20200720231237796.png)



------

해당 username/github.io에 들어가서, settings 클릭 

![image-20200720231333296](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20200720231333296.png)

아래처럼, GitHub Pages가 아직 disable되었다고 나옴

Source 선택:

Theme: 

아래와 같이 설정

![image-20200720231458102](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20200720231458102.png)

아래와 같이 repository name을 변경하면

![image-20200720231947766](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20200720231947766.png)

http://yjlab.github.io로 접속 가능해짐.



이후, git bash를 사용하여 위 site를 clone함

```bash
git clone https://github.com/yjlab/yjlab.github.io.git
```







원격 테마 설치:

1. Create/replace the contents of your `Gemfile` with the following: (이미 들어있음)

   ```
   source "https://rubygems.org"
   
   gem "github-pages", group: :jekyll_plugins
   ```

2. Add `jekyll-include-cache` to the `plugins` array of your `_config.yml`. (이미 들어있음)

3. `Start Command prompt with Ruby` 클릭

   ![image-20200720233518666](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20200720233518666.png)

4. Fetch and update bundled gems by running the following [Bundler](https://bundler.io/) command:

   ```
   bundle
   ```

5. Add `remote_theme: "mmistakes/minimal-mistakes@4.19.3"` to your `_config.yml` file. Remove any other `theme:` or `remote_theme:` entry.

6. git에 push

   ```bash
   git add .
   git commit -m"first commit"
   git push
   ```

   

You may also optionally specify a branch, [tag](https://github.com/mmistakes/minimal-mistakes/tags), or commit to use by appending an @ and the Git ref (e.g., `mmistakes/minimal-mistakes@4.9.0` or `mmistakes/minimal-mistakes@bbf3cbc5fd64a3e1885f3f99eb90ba92af84063d`). This is useful when rolling back to older versions of the theme. If you don’t specify a Git ref, the latest on `master` will be used.



**Note:** Your Jekyll site should be viewable immediately at [http://USERNAME.github.io](http://username.github.io/). If it’s not, you can force a rebuild by **Customizing Your Site** (see below for more details).

If you’re hosting several Jekyll based sites under the same GitHub username you will have to use Project Pages instead of User Pages. Essentially you rename the repo to something other than **USERNAME.github.io** and create a `gh-pages` branch off of `master`. For more details on how to set things up check [GitHub’s documentation](https://help.github.com/articles/user-organization-and-project-pages/).

![creating a new branch on GitHub](https://mmistakes.github.io/minimal-mistakes/assets/images/mm-gh-pages.gif)

You can also install the theme by copying all of the theme files[1](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#fn:structure) into your project.

To do so fork the [Minimal Mistakes theme](https://github.com/mmistakes/minimal-mistakes/fork), then rename the repo to **USERNAME.github.io** — replacing **USERNAME** with your GitHub username.

![fork Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/assets/images/mm-theme-fork-repo.png)

**GitHub Pages Alternatives:** Looking to host your site for free and install/update the theme painlessly? [Netlify](https://www.netlify.com/blog/2015/10/28/a-step-by-step-guide-jekyll-3.0-on-netlify/), [GitLab Pages](https://about.gitlab.com/2016/04/07/gitlab-pages-setup/), and [Continuous Integration (CI) services](https://jekyllrb.com/docs/continuous-integration/) have you covered. In most cases all you need to do is connect your repository to them, create a simple configuration file, and install the theme following the [Ruby Gem Method](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#ruby-gem-method) above.

### Remove the Unnecessary[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remove-the-unnecessary)

If you forked or downloaded the `minimal-mistakes-jekyll` repo you can safely remove the following folders and files:

- `.editorconfig`
- `.gitattributes`
- `.github`
- `/docs`
- `/test`
- `CHANGELOG.md`
- `minimal-mistakes-jekyll.gemspec`
- `README.md`
- `screenshot-layouts.png`
- `screenshot.png`

**Note:** If forking the theme be sure to update `Gemfile` as well. The one found at the root of the project is for building the theme’s Ruby gem and is missing dependencies. To properly setup a [`Gemfile`](https://github.com/mmistakes/minimal-mistakes/blob/master/docs/Gemfile) with the theme, consult the “[Install Dependencies](https://mmistakes.github.io/minimal-mistakes/docs/installation/#install-dependencies)” section.

## Setup Your Site[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#setup-your-site)

Depending on the path you took installing Minimal Mistakes you’ll setup things a little differently.

**ProTip:** The source code and content files for this site can be found in the [`/docs` folder](https://github.com/mmistakes/minimal-mistakes/tree/master/docs) if you want to copy or learn from them.

### Starting Fresh[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#starting-fresh)

Starting with an empty folder and `Gemfile` you’ll need to copy or re-create this [default `_config.yml`](https://github.com/mmistakes/minimal-mistakes/blob/master/_config.yml) file. For a full explanation of every setting be sure to read the [**Configuration**](https://mmistakes.github.io/minimal-mistakes/docs/configuration/) section.

From `v4.5.0` onwards, Minimal Mistakes theme-gem comes bundled with the necessary data files and will automatically use them via the [`jekyll-data`](https://github.com/ashmaroli/jekyll-data) plugin. So you no longer need to maintain a copy of these data files at your project directory.

You’ll need to create and edit these data files to customize them:

- [`_data/ui-text.yml`](https://github.com/mmistakes/minimal-mistakes/blob/master/_data/ui-text.yml) - UI text [documentation](https://mmistakes.github.io/minimal-mistakes/docs/ui-text/)
- [`_data/navigation.yml`](https://github.com/mmistakes/minimal-mistakes/blob/master/_data/navigation.yml) - navigation [documentation](https://mmistakes.github.io/minimal-mistakes/docs/navigation/)

### Starting from `jekyll new`[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#starting-from-jekyll-new)

Scaffolding out a site with the `jekyll new` command requires you to modify a few files that it creates.

Edit `_config.yml`. Then:

- Replace `<site root>/index.md` with a modified [Minimal Mistakes `index.html`](https://github.com/mmistakes/minimal-mistakes/blob/master/index.html). Be sure to enable pagination if using the [`home` layout](https://mmistakes.github.io/minimal-mistakes/docs/layouts/#home-page) by adding the necessary lines to **_config.yml**.
- Change `layout: post` in `_posts/0000-00-00-welcome-to-jekyll.markdown` to `layout: single`.
- Remove `about.md`, or at the very least change `layout: page` to `layout: single` and remove references to `icon-github.html` (or [copy to your `_includes`](https://github.com/jekyll/minima/tree/master/_includes) if using it).

### Migrating to Gem Version[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#migrating-to-gem-version)

If you’re migrating a site already using Minimal Mistakes and haven’t customized any of the theme files things upgrading will be easier for you.

Start by removing the following folders and any files within them:

```
├── _includes
├── _layouts
├── _sass
├── assets
|  ├── css
|  ├── fonts
|  └── js
```

You won’t need these anymore as they’re bundled with the theme gem — unless you intend to [override them](https://jekyllrb.com/docs/themes/#overriding-theme-defaults).

**Note:** When clearing out the `assets` folder be sure to leave any files you’ve added and need. This includes images, CSS, or JavaScript that aren’t already [bundled in the theme](https://github.com/mmistakes/minimal-mistakes/tree/master/assets).

From `v4.5.0` onwards, you don’t have to maintain a copy of the default data files viz. `_data/ui-text.yml` and `_data/navigation.yml` either. The default files are read-in automatically via the [`jekyll-data`](https://github.com/ashmaroli/jekyll-data) plugin.

If you customized any of these files leave them alone, and only remove the untouched ones. If done correctly your modified versions should [override](https://jekyllrb.com/docs/themes/#overriding-theme-defaults) the versions bundled with the theme and be used by Jekyll instead.

#### Update Gemfile[Permalink](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#update-gemfile)

Replace `gem "github-pages` or `gem "jekyll"` with `gem "jekyll", "~> 3.5"`. You’ll need the latest version of Jekyll[2](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#fn:update-jekyll) for Minimal Mistakes to work and load all of the theme’s assets properly, this line forces Bundler to do that.

Add the Minimal Mistakes theme gem:

```
gem "minimal-mistakes-jekyll"
```

When finished your `Gemfile` should look something like this:

```
source "https://rubygems.org"

gem "jekyll", "~> 3.5"
gem "minimal-mistakes-jekyll"
```

Then run `bundle update` and add `theme: minimal-mistakes-jekyll` to your `_config.yml`.

**v4 Breaking Change:** Paths for image headers, overlays, teasers, [galleries](https://mmistakes.github.io/minimal-mistakes/docs/helpers/#gallery), and [feature rows](https://mmistakes.github.io/minimal-mistakes/docs/helpers/#feature-row) have changed and now require a full path. Instead of just `image: filename.jpg` you’ll need to use the full path eg: `image: /assets/images/filename.jpg`. The preferred location is now `/assets/images/` but can be placed elsewhere or externally hosted. This applies to image references in `_config.yml` and `author.yml` as well.

------

That’s it! If all goes well running `bundle exec jekyll serve` should spin-up your site.

1. See [**Structure** page](https://mmistakes.github.io/minimal-mistakes/docs/structure/) for a list of theme files and what they do. [↩](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#fnref:structure)
2. You could also run `bundle update jekyll` to update Jekyll. [↩](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#fnref:update-jekyll)