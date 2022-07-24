# The y (sticky) regex flag in JavaScript

JavaScript allows a sticky `y` flag for regular expressions.

- See the `flags` parameter of the [RegExp() constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/RegExp)
for what this flag does.
- See [RegExp.prototype.sticky](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/sticky)
for more information.

Here is a basic example.

```javascript
(function() {
    const str = 'abbac-abbac';
    const regex = /ab|bac/y;

    // regex.test()
    console.log(regex.lastIndex, regex.test(str)); // match successfully and set regex.lastIndex accordingly
    console.log(regex.lastIndex, regex.test(str)); // match successfully and set regex.lastIndex accordingly
    console.log(regex.lastIndex, regex.test(str)); // fail to match and reset regex.lastIndex to 0
    console.log(regex.lastIndex);

    // str.match()
    console.log(regex.lastIndex, str.match(regex)); // same as above
    console.log(regex.lastIndex, str.match(regex)); // same as above
    console.log(regex.lastIndex, str.match(regex)); // same as above
    console.log(regex.lastIndex);
})();
```

Note that this flag does not seem standard in the field of regular expressions,
so it might not have an equivalence in other programming languages. Below are
examples where using this flag is possible but not necessary.

```javascript
(function() {
    // given a CSV string

    const csv = 'ab,"a",'; // 3 columns (but it doesn't matter)
    console.log(csv);

    // we want to match double quotes, commas and other characters

    const pattern = '[",]|.';

    // several approaches are implemented below

    (function() {
        console.log('--- using the global g flag (version 1): recommended ---');
        const regex = new RegExp(pattern, 'g');
        let match = null; // match data
        while((match = regex.exec(csv)) !== null) {
            console.log(match); // match[0] is the matched string
        }
    })();

    (function() {
        console.log('--- using the global g flag (version 2): all matches are saved in memory at once ---');
        const regex = new RegExp(pattern, 'g');
        const matches = csv.match(regex); // save all matches in memory at once
        if(matches !== null) {
            console.log(matches);
        }
    })();

    (function() {
        console.log('--- using the sticky y flag: this flag might not be available in other programming languages ---');
        const regex = new RegExp(pattern, 'y');
        var match = null; // match data
        while((match = csv.match(regex)) !== null) {
            console.log(match); // match[0] is the matched string
        }
    })();

    (function() {
        console.log('--- using no flag: requires extra substring() calculation ---');
        let _csv = csv; // because csv is const
        const regex = new RegExp(pattern);
        let match = null; // match data
        while((match = _csv.match(regex)) !== null) {
            console.log(match); // match[0] is the matched string
            // update _csv to avoid infinite loop: requires unnecessary substring calculation compared to the other approaches
            _csv = _csv.substring(match.index + match[0].length);
        }
    })();
})();
```
