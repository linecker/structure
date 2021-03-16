# Structure
Like paint for log files.

Somewhat similar to https://github.com/linecker/alps.

## Use case examples

* Line: split('\n')

* Fixed-length line header `D2012-11-19 16:24:51,001 QuartzWorker-1 ServiceRunner`
    * length 29
    * before third ' '
    * subtags date, time(ms), log-level

* Dynamix-length (since DEBUG and INFO have different length) line header `2012-11-19 16:24:51,001 DEBUG [email@example.com] QuartzWorker-1 ServiceRunner`

* Date field
    * Regular Expression `iso8601date(rx: '/^\d{4}-\d\d-\d\dT\d\d:\d\d:\d\d(\.\d+)?(([+-]\d\d:\d\d)|Z)?$/i');`

* Filtering
    * hide lines that contain a keyword
    * only show lines that contain a keyword
    
* Painting
    * Apply color to keywords

* Calculation
    * More margin if time-delta > certain something
    * Count occurences
    * Dynamic rules from first sight (I think postman can do this)

* Just fooling around ...

There are really just two directives: variable length-stuff use delimiters,
fixed-length stuff use length.

    length
    until


by subelements...
    
    main {
        lines;
    }
    
    lines {
        date;
        time;
        level;
        payload;
    }
    
    date {
        year;
        month;
        day;
    }
        
    part main, lines
        part lines, date time level payload\n
            date => year, month, day
                year
                    nnnn
                month
                    nn
                day
                    nn
            time => hour:minute:second,millisecond
                hour
                    nn
                minute
                    nn
                second
                    nn
                millisecond
                    nnn
            loglevel
                TRACE DEBUG INFO WARN ERROR FATAL
            payload
                
        ...
        {
            split_by_symbol('\n');
            struct line
            {
                split_by_range(1-10,12-23)
                struct date;
                struct time;
                struct level;
                struct payload;
            }
        }

## Thoughts, Requirements, Architecture
The input, quite abstractly, consists of a set of symbols. For log files, these 
symbols are typically arranged in lines and often have a header (e.g. a 
timestamp).

It would be nice to have a tool that can 1) assign names to certain structures
in such a set of symbols  (let's call this phase "tagging") and 2) apply styling
or code (e.g. bigger margin for lines with big time delta) to those structures. 

### Tagging

Do tags overlap?

It is OK do have structures within structures (e.g. date within line),
but structures can't overlap ("border mismatch"), e.g.:

    ok                      not ok (ignored)
    --------------          -------------- 
     ------ -----                ----
     -- - -                     ----
     
(This is different to alps, where there just was a sequential list of chunks.
Performance might suffer from this).

What's a good workflow for finding structures?

See something, select it, create a tag rule for it, test it. This works for 
1:1-matches, for tags with a certain format, we need something else (I can't
remember regexp so maybe something more EBNF or just code).

    for (int i=0; i<4; i++) {
        if (!number(0,255) || !symbol('.')) {
            return false;
        }
    }
    return true;

### Styling
CSS would probably be a good fit. If all of this is based in a browser view,
this might be the easiest part.

    line: {
        display: block;
        margin: 4px 0px;
    }

### Applying code

* Just playing around with syntax:

        structure line
        {
            --- tagging ---
            split: '\n';
            z: -1; /* expresses that z=0 (default) rules life within/under this */
            --- annotation ---
            display: block;
            margin: 8px;
            --- code ---
            lineCount++;
        }
    
* Field-references like in Excel?

### Implementation

I might need two layers, a bottom layer that does all the text actions and
a higher-level layer to implement library functionalities.


### Playground commands

* filter term
* color  term color

