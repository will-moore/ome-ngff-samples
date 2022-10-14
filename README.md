
# ome-ngff-samples

This is a page to list various OME-NGFF sample images from IDR. Deployed at https://idr.github.io/ome-ngff-samples/

# Development

To deploy locally, you'll need to comment-out this line in the Gemfile::

    gem "github-pages", group: :jekyll_plugins

Then you can run jekyll in a conda environment::

    conda create -n jekyll -c conda-forge rb-bundler c-compiler compilers cxx-compiler
    conda activate jekyll
    bundle install
    bundle exec jekyll serve

This will generate the html and serve it at http://127.0.0.1:4000/ but static files
such as JavaScript and CSS will be missing.
