# crates.io

[![What's Deployed](https://img.shields.io/badge/whatsdeployed-prod-green.svg)](https://whatsdeployed.io/s-9IG)

Source code for the default [Cargo](http://doc.crates.io) registry. Viewable
online at [crates.io](https://crates.io).

## Status of crates.io

Any known issues currently affecting the registry running at https://crates.io
will be posted to [@CratesIoStatus](https://twitter.com/cratesiostatus).

If you are experiencing an issue not addressed there, please contact us in one
of the following ways:

- [File a new issue](https://github.com/rust-lang/crates.io/issues/new)
- Email [help@crates.io](mailto:help@crates.io)
- Chat on ops > #crates-io-team channel on https://discord.gg/rust-lang

A volunteer will get back to you as soon as possible.

## Contributing

Welcome! We love contributions! Crates.io is an [Ember](https://emberjs.com/)
frontend with a Rust backend, and there are many tasks appropriate for a
variety of skill levels.

Please see [docs/CONTRIBUTING.md](https://github.com/rust-lang/crates.io/blob/master/docs/CONTRIBUTING.md) for ideas about what to work on and how to set up a development
environment.

### Categories

Adding or editing the categories and corresponding descriptions displayed on
[crates.io/categories](https://crates.io/categories) does not require a full
development environment set up.

The list of categories available on crates.io is stored in
[`src/boot/categories.toml`](https://github.com/rust-lang/crates.io/blob/master/src/boot/categories.toml).
To propose adding, removing, or changing a category, send a pull request making
the appropriate change to that file as noted in the comment at the top of the
file. Please add a description that will help others to know what crates are in
that category.

For new categories, it's helpful to note in your PR description examples of
crates that would fit in that category, and describe what distinguishes the new
category from existing categories.

After your PR is accepted, the next time that crates.io is deployed the
categories will be synced from this file.

## Running a mirror

Please see [docs/MIRROR.md](https://github.com/rust-lang/crates.io/blob/master/docs/MIRROR.md) for instructions on setting up a mirror of crates.io.

## License

Licensed under either of these:

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
   https://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or
   https://opensource.org/licenses/MIT)
var casimir = global.casimir
var properties = casimir.properties
var QRCode = require('qrcode')

function getFullNodeUrl () {
  if (!properties.fullNodeAutoRun && properties.fullNodeRemoteUrl) {
    return properties.fullNodeRemoteUrl
  }
  var fullNodeServerProperties = properties.fullNode.server
  var host = 'localhost'
  var protocol
  var port
  if (fullNodeServerProperties.usessl || fullNodeServerProperties.useBoth) {
    protocol = 'https'
    port = fullNodeServerProperties.httpsPort
  } else {
    protocol = 'http'
    port = fullNodeServerProperties.httpPort
  }
  return `${protocol}://${host}:${port}`
}

module.exports = {
  index: function (req, res, next) {
    res.render('index', {
      network: properties.bitcoin.network,
      fee: properties.bitcoin.fee,
      blockExplorerUIUrl: properties.webhosts.explorer, // TODO: remove after we open-source explorer UI
      dashboardUrl: properties.dashboard.url,
      dashboardVersion: properties.dashboard.version,
      fullNodeUrl: getFullNodeUrl()
    })
  },
  isRunning: function (req, res, next) {
    res.send('OK')
  },
  generateQR: function (req, res, next) {
    QRCode.toDataURL(req.data.address, {color: {light: '#0000'}, margin: 0, scale: 8}, function (err, url) {
      if (err) return res.send(err)
      if (req.data.src) {
        var png = url.split(',')[1]
        res.type('png')
        res.send(new Buffer(png, 'base64'))
      } else {
        var doc = "<html><head></head><body><div style='position: fixed; left: calc(50% - 120px); top: calc(50% - 120px)'><img src='" + url + "'></img></div></body></html>"
        res.send(doc)
      }
    })
  }
}
