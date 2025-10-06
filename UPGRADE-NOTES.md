# Jekyll Site Upgrade Notes

Date: October 5, 2025

## Upgrade Summary

This document describes the process of upgrading the Jekyll static site from older versions to more current libraries and dependencies.

## Ruby and Jekyll Upgrade

### Ruby Version

- Using Ruby 2.6.10
- Added Ruby version specification to Gemfile

### Jekyll Updates

- Upgraded Jekyll from ~3.6.3 to ~4.2.0 (highest version compatible with Ruby 2.6)
- Updated kramdown to 2.3.1
- Added kramdown-parser-gfm (required for Jekyll 4+)
- Replaced CodeRay with Rouge for syntax highlighting
- Added WebRick gem (required as it's no longer bundled with Ruby)

### Other Ruby Dependencies

- Updated activesupport to ~6.1.7
- Updated rake to ~13.0.6
- Updated thor to ~1.2.1
- Updated stringex to ~2.8.5

## Node.js Updates

### Node.js Requirements

- Updated Node.js requirement from 0.10.0 to 18.0.0+

### Grunt and Related Packages

- Updated grunt from 0.4.1 to 1.6.1
- Updated grunt-contrib-clean from 0.5.0 to 2.0.1
- Updated grunt-contrib-jshint from 0.6.3 to 3.2.0
- Updated grunt-contrib-uglify from 0.2.2 to 5.2.2
- Updated grunt-contrib-watch from 0.5.2 to 1.1.0
- Updated grunt-contrib-imagemin from 0.2.0 to 4.0.0
- Updated grunt-svgmin from 0.2.0 to 7.0.0
- Removed grunt-recess (deprecated)

## Configuration Updates

- Updated \_config.yml to use current Jekyll 4 syntax
- Removed deprecated pygments setting
- Replaced CodeRay configuration with Rouge syntax highlighter settings

## Known Issues

- Layout 'nil' warning for sitemap.xml (non-critical)
- Missing favicon.ico (non-critical)

## Installation Instructions

1. Install Ruby dependencies: `bundle install`
2. Install Node.js dependencies: `npm install`
3. Run the site locally: `bundle exec jekyll serve`

## Future Improvements

- Consider upgrading to Ruby 3.x for access to newer Jekyll versions
- Add favicon.ico or update references to favicon.png
- Fix sitemap.xml layout issue
