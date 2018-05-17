# Plugin List

A plugin is an independent module that can be added to Day.js to extend functionality or add new features.

By default, Day.js comes with core code only and no installed plugin.

You can load multiple plugins based on your need.

## API

#### Extend

* returns dayjs

Use a plugin.

```js
import plugin
dayjs.extend(plugin)
dayjs.extend(plugin, options) // with plugin options
```

## Installation

* Via NPM:

```javascript
import dayjs from 'dayjs'
import AdvancedFormat from 'dayjs/plugin/AdvancedFormat' // load on demand

dayjs.extend(AdvancedFormat) // use plugin
```

* Via CDN:
```html
<script src="https://unpkg.com/dayjs"></script>
<!-- Load plugin as window.dayjs_plugin_NAME -->
<script src="https://unpkg.com/dayjs/plugin/advancedFormat"></script>
<script>
  dayjs.extend(dayjs_plugin_advancedFormat);
</script>
```

## List of official plugins

### AdvancedFormat
 - AdvancedFormat extends `dayjs().format` API to supply more format options.

List of added formats:

| Format | Output           | Description                           |
| ------ | ---------------- | ------------------------------------- |
| `Q`    | 1-4              | Quarter                               |
| `Do`   | 1st 2nd ... 31st | Day of Month with ordinal             |
| `k`    | 1-23             | The hour, beginning at 1              |
| `kk`   | 01-23            | The hour, 2-digits, beginning at 1    |
| `X`    | 1360013296       | Unix Timestamp in second              |
| `x`    | 1360013296123    | Unix Timestamp in millisecond         |

## Customize

You could build your own Day.js plugin to meet different needs.

Template of a Day.js plugin.
```javascript
export default (option, c, d) => {
  const proto = c.prototype
  const oldFormat = proto.format
  // eslint-disable-next-line no-nested-ternary
  d.en.ordinal = number => `${number}[${number === 1 ? 'st' : number === 2 ? 'nd' : number === 3 ? 'rd' : 'th'}]`
  // extend en locale here
  proto.format = function (formatStr, localeObject) {
    const locale = localeObject || this.$locale()
    const utils = this.$utils()
    const str = formatStr || FORMAT_DEFAULT
    const result = str.replace(/Q|Do|X|x|k{1,2}|S/g, (match) => {
      switch (match) {
        case 'Q':
          return Math.ceil((this.$M + 1) / 3)
        case 'Do': {
          return locale.ordinal(this.$D)
        }
        case 'k':
        case 'kk':
          return utils.padStart(String(this.$H === 0 ? 24 : this.$H), match === 'k' ? 1 : 2, '0')
        case 'X':
          return Math.floor(this.$d.getTime() / 1000)
        default: // 'x'
          return this.$d.getTime()
      }
    })
    return oldFormat.bind(this)(result, locale)
  }
}

```