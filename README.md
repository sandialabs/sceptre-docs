SCEPTRE documentation.

View the docs here: https://sandialabs.github.io/sceptre-docs/

# Development info

The docs are hosted using GitHub Pages, and use the [Minimal](https://github.com/pages-themes/minimal) Jekyll theme for GitHub pages.

All pages are in Markdown format. Any `.md` files in the root of the repo will be rendered as a page and linked on the sidebar.

## Building locally

To preview changes to the docs locally, run the following commands:

```bash
# Install Ruby
sudo apt update
sudo apt install ruby-full

# NOTE: this may run into proxy issues if you're behind a web interception proxy. If that happens, reach out to a SCEPTRE developer for assistance.
gem install bundler
bundle install
bundle exec jekyll serve
```

Then connect to `localhost:4000`, or in VSCode, CTRL+Click on the URL it prints in the terminal. Changes to the markdown files will be automatically processed and docs will be regenerated in real-time while jekyll is running. However, changes to `_config.yml` and other files will requiring restarting the serve, by CTRL+C then re-running `bundle exec jekyll serve`.
