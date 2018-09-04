[![Astro Forms: Documentation](https://github.com/plummer/astro-docs/blob/master/assets/readme-header.jpg)](www.astroforms.com)

The documentation for Astro Forms, an iOS forms framework. Uses Vuepress docs.

## Development
1.  Ensure vuepress is set up correctly (<https://vuepress.vuejs.org/>)
2.  Run `vuepress dev src` to run the local server.

## Deployment
1.  Ensure vuepress is set up correctly (<https://vuepress.vuejs.org/>)
2.  Run `vuepress build src` to build a static site
3.  Commit any changes
4.  Run `sh deploy.sh` to publish the static site.

## Testing
1.  You can use any static server to test the built site locally
2.  For example using a node based server, run `npm install http-server -g`
3.	Then run `http-server src/.vuepress/dist`.
