# Hugo base for jelinek.website

This is the source code for my personal website. It is based on the [Hugo](https://gohugo.io/) static site generator.

## Development

To run the site locally, you need to have [Hugo](https://gohugo.io/) installed. Then, run the following command:

```bash
hugo server
```

## Cleanup and rebuild

If you have problems with the site not updating, you can try to clean up the modules and rebuild the site:

```bash
hugo mod clean
hugo server
```

## Deployment

The site is automatically built and deployed when changes are pushed to the `master` branch.