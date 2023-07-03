# MarkMaker

# Copied over from @jpetazzo's [container.training](https://github.com/jpetazzo/container.training/tree/main/slides) repository.

General principles:

- each slides deck is described in a YAML manifest;
- the YAML manifest lists a number of Markdown files
  that compose the slides deck;
- a Python script "compiles" the YAML manifest into
  a HTML file;
- that HTML file can be displayed in your browser
  (you don't need to host it), or you can publish it
  (along with a few static assets) if you want.



After making changes, run `./build.sh once`; it will
compile each `foo.yml` file into `foo.yml.html`.

You can also run `./build.sh forever`: it will monitor the current
directory and rebuild slides automatically when files are modified.

If you have problems running `./build.sh` (because of
Python dependencies or whatever),
you can also run `docker-compose up` in this directory.
It will start the `./build.sh forever` script in a container.
It will also start a web server exposing the slides
(but the slides should also work if you load them from your
local filesystem).
