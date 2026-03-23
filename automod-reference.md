# AutoModerator Reference

This document consolidates Reddit's AutoModerator documentation into a single offline reference. It covers writing rules, available inputs/outputs, limitations, common mistakes, YAML syntax, and a library of ready-to-use rules.

## Table of Contents

1. [Writing Basic Rules](#writing-basic-rules)
2. [Inputs, Modifiers & Outputs](#inputs-modifiers--outputs)
3. [Limitations](#limitations)
4. [Common Mistakes](#common-mistakes)
5. [YAML Syntax & Error Messages](#yaml-syntax--error-messages)
6. [Library of Common Rules](#library-of-common-rules)

---

## Writing Basic Rules

*(Introduction)*

This page is intended as an introduction to writing AutoModerator rules. If you are looking for complete information about AutoModerator's capabilities, see the full documentation page instead.

### Important Knowledge

The following information is extremely important and should be read and understood fully before proceeding: Rules must be separated by a line consisting entirely and exactly of ---. 3 hyphens, without any spaces before them. Rules that can cause the subject to be removed are checked first. Because of this, you should expect removal rules to take priority over all other types of rules. By default, rules that can cause the subject to be removed or reported will not be applied to moderators' posts. This behavior can be overridden, but keep it in mind, especially when trying to test. AutoModerator tries to avoid contradicting other mods' actions. It will not remove a post if it has already been approved by another moderator, or approve it if it has been removed by one.

### The Pieces of an AutoModerator Rule

An AutoModerator rule must consist of one or more checks (conditions that the comment or submission must meet) and one or more actions (things that will be performed on any posts that meet the conditions). Both checks and actions are defined by writing a line that has the name of the check or the action, followed by a colon, and then the value to set it to. For example, the line to define the action of removing a post looks like this: action: remove This sets the value of action for that rule to remove. A rule is made up by defining all of its checks and actions separately just like this. For example, here is a basic, complete rule to remove any submission that has the word "disallowed" in its title. Note that AutoModerator is not case-sensitive by default, so this rule will also apply to submissions with "Disallowed", "DISALLOWED", etc. in their titles:


```yaml
title: disallowed
action: remove
```


That's it. All AutoModerator rules are simply a combination of checks and actions, but some very complex rules can be built up from these pieces. In general, if you define multiple different checks on a rule, all of them must be satisfied in order to cause the actions to be performed. For example, here is a similar rule with two separate checks:


```yaml
title: disallowed
domain: youtube.com
action: remove
```


Now a submission with "disallowed" in its title will only be removed if it is also a link to YouTube. If it has "disallowed" in the title but goes to any other domain at all, it will not be removed because both of the conditions were not satisfied. If you instead want to remove a post if the word "disallowed" is in its title or it is a link to YouTube, that should be written as two separate rules instead (notice the --- line separating the two rules):


```yaml
title: disallowed
action: remove
---
domain: youtube.com
action: remove
```


### Checking for Any Item from a List

Of course, most rules do not apply to only a single word or domain. It is very common to write rules similar to "if the title contains any of these words...", so AutoModerator includes a way to define a list of possibilities, where the rule will be applied if any of the possibilities are found. For example, here is a rule that will set a submission's flair text to "Primary Color" for any submission with any of "red", "green", or "blue" in its title:


```yaml
title: [red, green, blue]
set_flair: Primary Color
```


To define a list of items, surround the entire list with square brackets, and separate items with commas.

### Reverse Checks

/ "must not match" Sometimes it is useful to be able to do a "reverse check", where instead of looking for the presence of something in particular, you want to check whether something is NOT present. For example, if a subreddit required every submission to be from YouTube, they would want a rule to remove any submission that is from any other domain. A check is reversed by putting a tilde ~ in front of it, so the rule for a YouTube-only subreddit would be something like:


```yaml
~domain: [youtube.com, youtu.be]
action: remove
```


### Author Checks

In addition to checking for conditions to be satisfied by comments or submissions themselves, you can also set conditions on the author of them. The simplest thing to check is the author's username. For example, say that your subreddit has two users named OfficialGuy and AnotherOfficialUser, and you want to give their submissions flair saying "Official Post" whenever they submit something. The rule would be:


```yaml
author: [OfficialGuy, AnotherOfficialUser]
set_flair: Official Post
```


There are also various other author checks supported related to characteristics of the user. For example, if you have a subreddit that has trouble with new accounts posting their blogs, you could write a rule that will remove any submission from tumblr or blogspot from accounts less than a day old:


```yaml
domain: [tumblr.com, blogspot.com]
author: account_age: < 1 day
action: remove
```


Notice that the account_age: setting is indented further than author: is. This is because the account_age check must be done "inside" the author check. For more information about what is possible with author checks, please see the section of the official documentation listing the other available ones.

### Checking for Type

One other check that can be extremely useful is to be able to specify which type of post you want a rule to apply to. For example, this rule would automatically remove all link submissions from users that have less than 5 comment karma, but will not affect any text submissions or comments they make:


```yaml
type: link submission
author: comment_karma: < 5
action: remove
```


### Posting Comments or Sending Modmail

In addition to directly taking actions on posts like approving, removing or reporting them or setting flair, one of the most common needs is to have AutoModerator leave a comment in response or send a modmail. This is done by simply defining comment: or modmail: as part of the rule. For example:


```yaml
domain: [youtube.com, youtu.be]
action: remove
comment: Sorry, your submission has been automatically removed.
```


Videos are not allowed in this subreddit. Or something like:


```yaml
title: [this subreddit, meta, mods, moderators]
modmail: This submission seems to be about the subreddit, a mod should probably check on it.
```


Note that a link to the post will automatically be included in a modmail, so the modmail from the above rule would include a link to the submission in question. Multi-line comments and modmails are also supported. To define text across multiple lines, the first line should be only |, and then all the lines to include in the text should be indented below, similar to putting age or karma checks inside author:. Here is a multi-line comment:


```yaml
domain: [youtube.com, youtu.be]
comment: |
    Sorry, your submission has been automatically removed.
    This subreddit does not allow videos to be posted.
    You might want to consider posting to /r/videos instead.
action: remove
```


There are also a number of "placeholders" that you can insert into comments and messages that will be replaced by the relevant information from the post. Here is an improvement of the modmail rule above that includes more information:


```yaml
title: [this subreddit, meta, mods, moderators]
modmail: |
    A submission that appears to be about the subreddit has been made by /u/{{author}}. **Title:** {{title}} When this rule matches a submission, {{author}} and {{title}} will be replaced in the modmail by the author's username and the post's title respectively.
```


---

## Inputs, Modifiers & Outputs

### Introduction

AutoModerator acts on content when it's submitted, edited, reported (if the reports check is present); it can also check and act on a post from detecting a comment on it (but not on the author of the post).

AutoModerator rules are made up of four parts:

- **Inputs** — the criteria that will be used to make a decision
- **Input Modifiers** (optional) — any change from the default logic that is used to evaluate the inputs
- **Outputs** — action(s) taken if all of the input criteria are met
- **Behavioral Modifiers** (optional) — any change from the default behavior of a particular output action (see 2nd to last section)

### Simple Example

with the word video in the title must be to YouTube


```yaml
type: link submission
title (includes): "video"
~domain: [youtube.com, youtu.be]
action: remove
comment: Your post was automatically removed.
```


All link submissions with the word video in the title must be to YouTube. title + domain - Input. We're checking to see if the title includes the text "video" and if the domain matches "youtube.com" or "youtu.be". (includes) + ~ - Modifier. Applying the includes modifier to the title check means that we're looking for the text "video" even if it's part of a longer word like "videos" or "musicvideo". Applying the ~ modifier means that we're actually checking for domains that don't match "youtube.com" or "youtu.be" action: remove + comment - Output. Remove the post and leave a comment on the post. Inputs, Modifiers, and Outputs can be combined in almost any way to design custom rules.

### Inputs

can use to make its decisions For the most up-to-date list of checks see the full doc

#### Modifiable

These inputs have to do with matching some type of text, and can be modified with Modifiers.

some type of text, and can be modified with Modifiers. title - Post title domain - Linked domain (link posts only) url - Linked url (link posts only) author - User's name (author modifiers can only be used in the indented name) body - Text of a self-post or comment flair_text - The text of a flair (of a post or a user, depending on the indentation) flair_css_class - the css class of a flair (same) flair_template_id - the ID of a flair (same) id - The ID of the content. It's the 5-7 letters and numbers that come after /comments/ in the url. media_author - User's display name on the linked media site (YouTube user, etc.) (link posts only) media_title - Title of linked media (YouTube video title, etc.) (link posts only) media_description - Description of linked media (YouTube video description, etc.) (link posts only)

#### Non-Modifiable

The following inputs do not relate to text matching and therefore cannot be modified with Modifiers.

to text matching and therefore cannot be modified with Modifiers: type - "submission" or "comment" or "both" reports - a minimum number of reports that an item has received is_top_level - true for comments that are top-level, false for comments that are not. (Comments only) priority - specifies an order in which rules should be checked. (Rules with actions that remove a content are always checked before all other rules, regardless of priority) Conditions of a user's account - subgroup these under author. If you also want to check username, use name (also subgrouped under author) account_age - age of user's account (in days by default) post_karma - the user's link karma comment_karma - the user's comment karma combined_karma - the user's combined karma (link karma + comment karma) has_verified_email - whether the user has a verified email address (true or false) is_gold - whether the user has reddit gold or not (true or false) is_submitter - whether the author of the comment is the OP of the post (true or false, applies to comments only) is_contributor - whether the user is in the Approved Users list or not (true or false) is_moderator - whether the user is a moderator in the subreddit or not (true or false) satisfy_any_threshold - whether the rule should act if any karma or account age check is matched even if the others don't (true or false, false by default, only applies to karma and account age checks)

### Input Modifiers

Modifiers are optional. They change how AutoModerator evaluates text-based inputs.

. They change how AutoModerator evaluates text-based inputs. case-sensitive - makes it so that the match is dependent on capitalization matching as well. ~ - makes it so the rule will act only if that check does not match. regex - interprets the match values as regular expressions instead of literal strings The following modifiers are used for text matching. Only one at a time may be specified for a given input. To see which inputs default to which matching mode, see the full documentation. full-exact - only match if the entire string being checked matches the pattern exactly. full-text - only match if the text of the entire string matches the pattern exactly (allows extra spacing/punctuation on either side). includes-word - match if the text includes any of the full words specified in the pattern. For example, a pattern of base will match against "all your base", but not "I like baseball". includes - match if the text includes the pattern at all (not only full words). starts-with - match if the text starts with the pattern (not only full words). ends-with - match if the text ends with the pattern (not only full words).

### Outputs

If all of the Input criteria are satisfied (according to any Modifiers specified), AutoModerator will perform the Outputs.

(according to any Modifiers specified), AutoModerator will perform the Outputs. Possible Outputs: action: - Perform a moderator action. Only one of these may be specified, as Approved, Spammed, Removed, and Reported are all mutually exclusive states. approve - approves the content. (only performed on items in the modqueue. Never performed on content by users with reddit-wide shadowbans, unless the user is specified by name with the user input) remove - removes the content. filter - removes the content and sends it to the modqueue. spam - removes the content as spam (to train the subreddit's spam filter). report - reports the content, adding it to the modqueue. comment - leaves a distinguished comment on the item modmail - sends a message to the subreddit moderators message - sends a private message to the user set_flair: ["text", "css class"] - set the text and css class (optional) of a flair (of a post or a user, depending on the indentation) Or formatted like this to also control the flair design on New Reddit Note that the text and css_class are optional when using template_id which includes the default text and css class of that flair set_flair: template_id: text: css_class: set_nsfw - Tags a post as NSFW set_sticky - Stickies the post to the top of the subreddit (not to be confused with comment_stickied) set_spoiler - Tags a post as spoiler set_contest_mode - Sets contest mode on the post (randomized comment display order and hidden votes) set_original_content - Tags a post as OC set_suggested_sort - Sets the sorting of comments on a post (best, new, qa, top, controversial, hot, old, random and blank) set_locked - Locks a post (not to be confused with comment_locked) comment_stickied - Stickies AutoMod's comment if it leaves one comment_locked - Locks AutoModerator's comment if it leaves one

### Behavioral Modifiers

Change the way AutoModerator acts on an item.

acts on an item overwrite_flair - Indicates whether or not AutoModerator should override any pre-existing post flair or user flair. Defaults to false if not specified. moderators_exempt - If set to false it changes the behavior for rules with the actions: remove, filter, spam, and report (by default, mods are immune from those actions). If set to true it changes the behavior for rules without the mentioned actions, for example a rule that only comments based on a detected keyword. action_reason - Changes behavior on the actions: remove, filter and report. Sets text to accompany a report or removal. Defaults to nothing if not specified. modmail_subject - Changes behavior on modmail. Sets a subject line. Defaults to "AutoModerator Notification" if not specified. message_subject - Changes behavior on message. Sets a subject line. Defaults to "AutoModerator Notification" if not specified.

### Variables

AutoModerator's comments, modmail, and messages can include variables.

, and messages can include variables, allowing the comment/modmail/message to give information unique to that specific post. {{permalink}} - permanent link to content. Will be automatically included at the top of a modmail or message if not included elsewhere. {{title}} - the title of a submission that was matched. (submissions only) {{body}} - full text of the item. (comments and text submissions only) {{domain}} - the domain of a submission (link submissions only) {{url}} - the url of a submission that was matched. (link submissions only) {{kind}} - "comment" or "submission" {{subreddit}} - the name of your subreddit. (Does not include /r/) {{author}} - the user whose content triggered the rule. (Does not include /u/) {{media_user}} - the user name on the linked media site (YouTube user, etc.) (link submissions only) {{media_title}} - the title on the linked media site (YouTube video title, etc.) (link submissions only) {{media_description}} - the description on the linked media site (YouTube video description, etc.) (link submissions only)

---

## Limitations

### Explanation

To understand why AutoModerator can't do certain things, it's helpful to understand first how it operates.

why AutoModerator can't do certain things, it's helpful to understand first how it operates. AutoModerator does the following, in a never-ending loop: Load the /new, /comments, and /about/modqueue pages of the subreddits that it moderates Check for anything that's freshly appeared On items that have freshly appeared, execute any actions as dictated by that subreddit's custom rules. Content submitted by a subreddit moderator or approved by another moderator is ignored. Return to step 1. To sum up: AutoModerator checks new content, makes a decision, acts on that decision (typically within just a few seconds of a post/comment being submitted), and then moves along, never to return.

### Things AutoModerator Can't Do

Delay the checking of content Re-check content except when something has been reported or when something is edited Delay an action Detect reposts Make decisions based on vote score Make decisions based on any piece of content other than the submission or comment that is currently being examined (The only exception is that the properties of the parent submission may be considered when evaluating comments.) Make decisions based on link flair or user flair being added, modified, or removed (Flair may be considered when evaluating a submission or comment, but flair updates do not lead to a re-check.) Perform a randomized action Perform an action that requires config, access, mail, or full moderator permissions Do math Do image analysis Take any kind of input from any 3rd-party source, including (but not limited to): YouTube channels, social media, content on other subreddits, AutoMod configs of other subreddits, wiki pages other than the AutoMod config page. Make decisions based on a user's post/comment history, or a link's post history If you need to automate a moderator action that requires math, randomization, analysis of user history, or any of the above types of actions, try /r/RequestABot or /r/Devvit. There are also Developer Platform apps which may suit such needs listed on this wiki page or on this ModSupport post.

---

## Common Mistakes

### Editing with new Reddit

New Reddit / Desktop redesign outputs nearly useless error messages like "Something went wrong" or "Unsupported Media Type" when there is a failure saving AutoModerator configuration. Use classic "old" Reddit (https://old.reddit.com) when editing AutoModerator configuration to receive a somewhat more useful error message that might help you debug any errors.

### Using LLMs to Write Code

Large Language Models like ChatGPT often guess or make up terms or syntax that do not work with AutoModerator. Sometimes you will find this out when your code does not save in the config page. Other times you will find this out much later when your code is doing something extra that you do not want it to do, or your code is not actually doing the the thing you wanted it to do in the first place. If you are savvy with LLMs, you will get better results if you train your session with AutoModerator-specific documentation. However, you still may not save time doing that compared to reading the documentation yourself. If you are not savvy with LLMs, you will get better results by reading the Full-Documentation page and the rest of our wiki here, or by sharing your goals and code with other community members to see in your post.

### Testing Rules as a Moderator

A common mistake is expecting all rules to apply to moderators, especially when testing rules. Moderators are exempt by default from rules that result in any kind of removal or a report. To test removal or report rules, either use a non-moderator test account or add moderators_exempt: false so the rule will apply to moderators.

### Overusing `includes` Modifier

Using includes instead of includes-word can result in many false positives, especially when matching short keywords. includes-word is the default behavior for body and title checks and generally works best when matching written language (especially words and phrases). See the "matching modifiers" section of the full documentation for more information. For example, let's say you want to match the word "cow" in singular, plural, and as part of words like "cowboy", "cowgirl", and "cowbell". Using body (includes): ['cow'] would allow matching all of those, but it's not the best way to do this because it would also match unrelated words like "scowl" and "coward". A better approach to match variations of a base word without an exhaustive list is to use a regex check. For this example, body+title (regex): ['cow(bell|boy|girl)?s?'] will only match the desired words. In general, includes is somewhat safer to use on longer keywords, but the main situation where includes is useful is combined with regex when writing rules targeting characters or sequences of characters rather than words or phrases.

### Quoting

Any text in AutoModerator rules that contains special characters which have a specific meaning in AutoModerator configuration (based on the YAML format) needs to be quoted using single quotes or double quotes. For example, the : (colon) character has a very specific meaning in AutoModerator configuration files so if you are trying to match a :, you need to quote that part of your expression. To keep things simple, it's easiest to single quote all expressions, especially regex. For the comment, modmail, and message fields, using the multi-line format is another option (instead of using quotation marks). There are also some less common cases listed in the full documentation.

### Smart Quotes

Many devices and applications use "smart punctuation" for quotes (also called "smart quotes" or "curly quotes") and dashes. Smart quotes in particular will cause problems when writing AutoModerator configuration. You need to use straight vertical quotation marks (" or ') instead of smart quotes. If you're having this issue, you may want to change your device or application settings to disable smart punctuation. Quoting quotes If you need to include a single quote as part of an expression that is already wrapped in single quotes, you need to put two single quotes in a row like body: ['I''m tired']. The perhaps better option is to use a regular expression like body (regex): ['I[\x27\x2019]m tired'] which has the advantage of allowing matches on different characters used for the apostrophe: U+0027 (apostrophe) and U+2019 (right single quote, the smart quote version).

### Indentation

Indentation should be consistent: use the same number of spaces at the start of each line, in a sub-group, comment text, etc. Indentation should be done with spaces and not tabs. Rules need to have a separator line between them which consists of 3 dashes --- without any indentation.

### Rule Ordering

Rules that result in a removal will always run before any other type of rule (regardless of priority) and if a submission or comment is removed then no other rules will act on it. All other types of rules run from top to bottom unless a priority is specified. Note that if you have a tiered set of rules, for example rules that change the user flair's text in sequence, then the rules should be ordered in reverse (with the last rule in the series being at the top) or have decreasing priority values, otherwise once the first rule runs and changes the flair then the 2nd rule will run and act based on the new flair, and so forth until the last rule.

### Multiple Expression Rules

Rules that have 2 or more separate checks will only run if all those checks are matched. For example, if we have a title check and a separate body check then one item needs to be matched from the title check and one item needs to be matched from the body check. Note that a title+body check is a single check where only one item needs to be matched from either of those fields, not both. Using the same check multiple times (for example 2 title+body checks) will only run the last instance of it. See Match Multiple Keywords from the library for some example solutions.

### Wrong Checks

Links The url and domain checks only apply to the URL of a link submission. To also match the text content of self posts (text posts) and comments, you need to add title and body to the check. For example, to block badsite.com the line would be body+domain+title: ['badsite.com'].

### Wrong Modifiers

By default, flair checks are done as full-exact which means that the whole text needs to match. We need to specify includes or includes-word when the flair contains more text/emojis other than the keywords/phrases that are being checked. This also applies to checks in sub-groups like author, but it can be disabled for karma and account age checks by indenting satisfy_any_threshold: true under author - satisfy_any_threshold only applies to karma and account age checks. Any additional check under author (name, flair_text, has_verified_email, etc.) will always be performed even when satisfy_any_threshold is set to true Special characters Matching special characters Some special characters such as emojis cannot be directly entered into AutoModerator configuration. To use them in a check or output them in a text, you need to use their Unicode value (in regex) or their HTML code. For checks: paste the character into https://unicode-table.com/ to find its Unicode value. Then you can write it as a Unicode escape: Characters from U+00 to U+FF can be written as a 2-digit hexadecimal code. For example, U+0027 (single quote or apostrophe) can be written as \x27. Any Unicode from U+0000 to U+FFFF can be written as a 4-digit hexadecimal code. For example, U+2705 can be written as \u2705. Any Unicode from U+0000 to U+FFFFFFFF can be written as an 8-digit hexadecimal code (and that is the only syntax for any Unicode characters higher than U+FFFF. For example, U+1F600 must be written as \U0001F600. Always "pad" the left side of expression so the length of the hexadecimal is 8 digits. Unicode expressions also work in regex character classes and character class ranges.

### Examples

:


```yaml
---
title (regex): ['\U0001F600', '\u2705']
---
    or
---
title (regex): ['[\u2705\U0001F600]' --- Outputting special characters To output a special character in a comment or message, paste the character into https://unicode-table.com/ and copy the HTML code. For example, for this emoji, you can use comment: "Hi &#128522;"
```


. To include a special character in flair text, you need to use the custom emoji code from mod tools like :cake: - set_flair: "Verified :cake:". Things AutoModerator Can't Do There are things that AutoModerator can't do like: looking at votes on posts/comments, delaying a check/action (e.g., acting on a flair that's added/changed after posting, or removing a matched post after one day), looking at parent comments, looking at other comments in a post (or lack thereof), only allowing specific types of posts on specific days, etc.

### Regex Mistakes

Double-quoted regex requires double escaping of special characters ("domain\\.com"), while single-quoted regex requires single escaping ('what is it\?') Confusing parenthesis groups and character classes Trying to use match placeholders or backreferences without realizing that regex are always concatenated together, joined by |, and surrounded by parentheses (e.g., off-by-one errors)

### Practices / Styling

Checks are case-insensitive by default and so there's no need to account for different cases of the same keyword/phrase Not sorting long rules alphabetically for easier maintenance Not testing expressions on normal text to look for false positives Not adding an action_reason for every action Not using [{{match}}] in action_reason strings for highlighting of matches for Toolbox users Overuse of modmail when filter is sufficient In regex: Too much .* when it's not needed or matching too far Using . within domains and hostnames in a regex without realizing . means "any character" It's preferable to use single-quotes when using regex because we escape characters more frequently than we use apostrophes, and it allows us to test the regex as it's written in one of the regex testing sites. (Remember that the wrapping quotes aren't part of the regex and so don't copy them when testing.) Not using \b enough around words when doing includes rules Using \b when it's not needed (i.e., start or end of regex that that is implicitly or explicitly includes-word) Not handling suffixes, plurals, etc. properly with regex Examples Dashes Individual AutoModerator rules need to be separated by --- (three dashes) on its own line. Failure to do this will cause multiple rules to be treated as a single rule, resulting in unexpected behavior. Correct:


```yaml
---
domain: [bannedsite1.com, bannedsite2.com]
action: remove
---
domain: [spamsite1.com, spamsite2.com]
action: spam
---
    Indentation Indentation is more important than most people realize. Indentation tells AutoModerator how certain types of data are structured. Improper indenting will prevent AutoModerator from being able to correctly process your configuration. The main indentation (for lines such as type/title/action/etc.) isn't a requirement, but it is helpful for visual separation of the different rules, and it keeps the code's format as a Code Block when submitting it in a post/comment (when it's indented with 4 spaces). Any amount of indentation is acceptable (usually 2 or 4 spaces) as long as it's consistent among the specific block/sub-group (author / parent_submission / comment text / etc.). The
---
    lines shouldn't be indented Here's the correct indentation for almost everything that can go in an AutoModerator rule. Note that most of these lines are missing the "something" on the right side of the : character to be part of a working rule. This section is only showing the proper indentation!
---
type: title: ['list', 'of', 'expressions']
body: - 'keyword1'
# "Dictionary"-type lists of expressions - 'keyword2'
# should be indented
```



```yaml
domain: url:
author: name:
# These
comment_karma: # go
post_karma: # inside
combined_karma: # the
account_age: # "author:" has_verified_email:
# block
satisfy_any_threshold: set_flair: template_id:
flair_text: flair_css_class:
parent_submission: set_flair:
# These
flair_text: # go
flair_css_class: # inside
id: # the
title: # parent_submission
domain: # block
body: media_author:
media_author_url: media_title:
media_description: comment:
    | This is a multi-line comment. Notice how all the comment text lines are indented additionally and by the same amount.
message_subject: message: | This is a multi-line private message to the user. Notice how all the message text lines are indented additionally and by the same amount.
modmail_subject: modmail: | This is a multi-line modmail to the moderation team. Notice how all the modmail text lines are indented additionally and by the same amount.
action: action_reason:
---
```


### Error Messages

These are some of the errors you can get while trying to save AutoModerator configuration through old Reddit:

of the errors you can get while trying to save AutoModerator configuration through old Reddit: Error Issue "YAML parsing error in section #: mapping values are not allowed here" No quotation marks around text with some of these special characters like a colon: comment: Make sure to read the rules: https... "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '<block mapping start>'" Line indentation is inconsistent "YAML parsing error in section #: while scanning a quoted scalar [...] found unexpected end of stream" Missing a single-quote at the end of an item: body: 'keyword "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '<scalar>'" Separator line (---) is indented, or: A non-doubled apostrophe in an item that's wrapped in single-quotes: 'I'm', or: Missing a double-quote at the end of an item: body: "keyword "YAML parsing error in section #: while scanning a simple key [...] could not found expected ':'" Not enough dashes in a separator line / wrong type of dashes, or: No space after a colon: body:"keyword", or: No colon after a check/etc.: body "keyword", or: Missing an indented dash at the start of a line in a check with a "dictionary"-type list (i.e. a vertical list instead of a horizontal list separated by commas) "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '<block sequence start>'" A line with a different indention than the rest in a check with a "dictionary"-type list (i.e. a vertical list with dashes instead of a horizontal list separated by commas) "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '-'" A line with no indentation in a check with a "dictionary"-type list (i.e. a vertical list with dashes instead of a horizontal list separated by commas) "YAML parsing error in section #: while parsing a flow sequence [...] expected ',' or ']', but got '<scalar>'" Missing a double-quote at the end of an item:


```yaml
body: ["keyword], or: Missing a single-quote at the end of an item: body: ['keyword1, 'keyword2'], or: Missing a closing bracket: body: ['keyword1', 'keyword2', or: Missing a comma in a list of quoted items bracket: body: ["keyword1" "keyword2"]
```


"YAML parsing error in section #: while parsing a block node expected the node content, but found '?' Too many dashes in a separator line "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found ','" Missing an opening bracket: body: 'keyword1', 'keyword2'] "Can't use combined_karma on this type in rule:" Profile-related check line isn't indented under author "YAML parsing error in section #: while scanning a block scalar [...] expected a comment or a line break, but found '[...]' Non-quoted "bigger than" karma/account age check: comment_karma: > 5, or: First line of a multi-line comment/message/modmail format isn't indented under the function: `comment: "YAML parsing error in section #: unacceptable character [...]: special characters are not allowed" Including characters that are unsupported by Automod's configuration page (like emojis) "Invalid search check: [...] in rule:" No parenthesis for the specified check modes: body includes: "keyword", or: Missing underscore in a function's multi-word name like comment stickied and is top level "invalid value for type: Submission in rule:" Capitalized value in a type check: type: Submission "YAML parsing error in section #: while scanning a double-quoted scalar [...] found unknown escape character" Single-escaping in a double-quoted regex: body (regex): "\w+" "Generated an invalid regex for [...]: multiple repeat at position # in rule:" Double asterisks in a regex: '.**' No error but it won't save Listing multiple types which isn't supported: type: ["link submission", "comment"], or: Including unsupported characters in the "reason for revision" field when saving.

---

## YAML Syntax & Error Messages

The purpose of this page is to provide an overview for just the YAML implementation and syntax requirements for AutoModerator, without necessarily detailing every individual check or action. The bulk of the Syntax section is copied from the Full Documentation Page, with subheadings added and a few lines from the Karma/age threshold checks and sub-groups sections. The Checks and Actions subsection is copied from the Writing Basic Rules Page. The Custom Match Subject Suffixes section is copied from this post in August 2014, and has not yet been added to the Full Documentation page. The Error Messages section is copied from https://www.reddit.com/r/AutoModerator/wiki/common_mistakes (by /u/001Guy001)

### Custom Match Subject Suffixes

### YAML Terms

AutoModerator rules are defined using YAML.

using YAML, so for full details about allowable syntax you can look up examples or the YAML specification (kind of a difficult/technical document). Some of the most important things to know for AutoModerator specifically are discussed in later sections on this page below. These YAML term definitions may help, but are not necessary to understand except when encountering Error Messages. Italic text is copied from the YAML specification. Node - literally, a single native data structure, which can have content that is a scalar, sequence or mapping. Scalar - any string or number. (more literally, an opaque datum that can be presented as a series of zero or more Unicode characters) Sequence - a list of strings or numbers. (more literally, an ordered series of zero or more nodes.) Key/Value Pair - an assignment of a Sequence or Scalar (Value) to another Scalar (Key). Mapping - literally, an unordered set of key/value node pairs, with the restriction that each of the keys is unique. Also known as Dictionaries. Structural Productions and Block Style Production - essentially, indentation denotes structure. Document - an independent set of Nodes containing content Stream - a set of Documents separated by markers In the context of AutoModerator: a Key would be any Check or Action. The Value would be what you assign to it. a Mapping is all of the Checks or Actions (and their respective assignments) on the same level (either top-level or sub-group). the Indentation of each line is important for proper structure of Mappings, Lists, Multi-Line Strings, and Sub-groups. Mismatched indentation often leads to syntax errors. a Document is a full AutoModerator rule. The Stream is the full AutoModerator config page content. Additional relevant YAML terms are inserted into the sections below (using italics and parentheses).

### Rule Boundary

Rules must be separated by a line starting with exactly 3 hyphens: `---`

by a line starting with exactly 3 hyphens - ---

### Comments

Comments can be added by using the `#` symbol.

symbol. Generally everything after a # on a line will be treated as a comment and ignored, unless the # is inside a string or otherwise part of actual syntax. Some common reasons to add comments include to: give a rule a title / name for reference explain who wrote the rule, or where it came from add context to why/when a rule was added or changed, or what it is trying to do deactivate a rule without clearing it from the config, in case you want to find it again in the future

### Checks and Actions

An AutoModerator rule must consist of one or more checks and one or more actions.

of one or more checks (conditions that the comment or submission must meet) and one or more actions (things that will be performed on any posts that meet the conditions). Both checks and actions are defined by writing a line that has the name of the check or the action, followed by a colon, and then the value to set it to. For example, here is a basic, complete rule to remove any submission that has the word "disallowed" in its title:


```yaml
title: disallowed
action: remove
```


All AutoModerator rules are simply a combination of checks and actions, but some very complex rules can be built up from these pieces. In general, if you define multiple different checks on a rule, all of them must be satisfied in order to cause the actions to be performed. For example, here is a similar rule with two separate checks:


```yaml
title: disallowed
domain: youtube.com
action: remove
```


If you instead want to remove a post if the word "disallowed" is in its title or it is a link to YouTube, that should be written as two separate rules instead (notice the --- line separating the two rules):


```yaml
title: disallowed
action: remove
---
domain: youtube.com
action: remove
```


### Strings

Strings do not generally need to be quoted, but it is usually safest to put quotes around them.

(as in plain style Flow Scalar), but it is usually safest to put either single or double quotes around a string (as in quoted style Flow Scalar), especially if it includes any special characters at all. For example, the quotes here are unnecessary but encouraged: title: ["red", "blue", "green"] Converts If you do not use quotes, there are certain types of strings that the YAML parser will try to automatically convert, which can result in unexpected behavior. In general, these include strings of numbers that start with 0 or 0x, strings that consist of only numbers and underscores, and the words true, false, on, off, yes, no. If in doubt, it is always safest to use quotes. Similarly, this also applies for threshold checks: Note that due to the > symbol having a special meaning in YAML syntax (for folded style Block Scalars), you must put quotes around a greater-than check, but it is not necessary for less-than checks. For example, a check to see if the author has more than 10 post karma would have to be written as:


```yaml
author: post_karma: '> 10'
```


### Escaping

When defining regular expressions, single quotes are highly encouraged to avoid double-escaping.

inside a search check, you should always surround the regular expression with quotes, but single quotes are highly encouraged. This avoids needing to double-escape. For example, this check includes the exact same regex twice, but the double-quoted version requires double-escaping all the special characters: title (regex): ["\\[\\w+\\]", '\[\w+\]'] Note that if you need to include a single quote inside a single-quoted string, the way to do so is by typing two single quotes in a row, not with a backslash. For example: 'it''s done like this'.

### Multi-line Strings

Multi-line strings use the pipe `|` character.

as well, this is used almost exclusively for defining multi-line comments to post or messages/modmails to send. The syntax for a multi-line string (aka literal style Block Scalar) is to have a single pipe character (|) on the first line, and then indent all lines of the multi-line string below and inside. For example: comment: | This is a multi-line comment. It has multiple lines. You can use **markdown** inside here too.

### Lists

AutoModerator includes a way to define a list of possibilities using square brackets.

similar to "if the title contains any of these words...", so AutoModerator includes a way to define a list of possibilities, where the rule will be applied if any of the possibilities are found. Lists of items can be defined in two different ways. The most compact method (aka Flow Sequence) is inside square brackets, comma-separated: title: ["red", "green", "blue"] The other method (aka Block Sequence) is by indenting the list of items below, with a hyphen at the start of each line. This format is often better for longer or more complex items, or if you want to add a comment on individual items: title: - "red" # like apples - "green" # like grapes - "blue" # like raspberries Both formats are exactly the same from AutoModerator's perspective, but one can often be far easier to read than the other.

### Sub-groups

AutoModerator rules support "sub-groups" of checks and actions.

"sub-groups" of checks and actions that apply to things that are related to the main item being targeted by the rule. Checks and actions inside the sub-group should be indented below and inside it. (Essentially, a new mapping) For example, here is a rule that utilizes two sub-groups to set a submission's flair text to "Possible Repost" if a user with the "trusted" flair css class makes a top-level comment inside it including the word "repost":


```yaml
type: comment
body: "repost"
is_top_level: true
author: flair_css_class: "trusted"
parent_submission: set_flair: "Possible Repost"
```


### Duplicate Checks

Avoid defining the same thing twice inside a particular rule.

the same thing twice inside a particular rule. This will just end up with the second definition overwriting the first one. (In YAML terms, this is because each mapping requires the keys to be unique.) For example, a rule like this will end up only affecting youtube submissions and not imgur:


```yaml
domain: imgur.com
domain: youtube.com
action: remove
```


Custom Match Subject Suffixes (aka no more need for body+body shenanigans) Old joint method It's always been possible to do multiple checks against the same field, but it's always been pretty unwieldy to do and required using an ugly hack to get a different "name" for the additional checks. For example, if someone wanted to remove a comment containing both "red" and "blue", this would generally be done as:


```yaml
type: comment
body: red
body+body: blue
action: remove
```


The body+body format where field names are joined by a + was actually intended for being able to check multiple fields to see if any match, such as doing title+body+domain: to check all three of those for something. However, since YAML doesn't allow defining exactly the same check multiple times (the later definitions will just overwrite the earlier ones), this also became used as a hacky way to be able to write multiple checks on the same field by getting different "names" that still only check the same fields. In some cases it got extremely ugly, getting up to body+body+body and further. New suffix method You can now write your own custom "names" by appending # + any suffix to a check. So for example, instead of the above, that rule could now be written as:


```yaml
type: comment
body: red body#2: blue
action: remove
```


Numbers are probably the simplest way to use this, but you can actually put anything you like after the # to make that check's name unique. So it can also be used as a sort of comment to make things simpler to read, if you want to do something like:


```yaml
type: comment
body: [red, blue, yellow]
```


body#shapes: "(square|circle|triangle)s?" modifiers: body#shapes: regex action: remove Editor's Note: The modifiers syntax was also updated later in when AutoModerator was integrated, so the last example could now be written as:


```yaml
type: comment
body: [red, blue, yellow]
```


body#shapes (regex): "(square|circle|triangle)s?" action: remove

### Anchor and Alias

AutoModerator supports Anchors and Aliases for re-using values.

Anchors and Aliases. Together, these allow a YAML value to be re-used across a single YAML document, ie, a string or list could be defined and used in more than one check or action, but only within a single AutoModerator rule. For Example:


```yaml
type: submission
title: - Something - &test remove - Whatever
```



```yaml
action: *test
moderators_exempt: false
```


That sets &test to remove and then uses it as the action later while also checking for it in the title Example by Deimorz.

### Error Messages

These are some of the errors you can get while trying to save AutoModerator configuration through old Reddit:

you can get while trying to save AutoModerator configuration through old Reddit: Error Issue "YAML parsing error in section #: mapping values are not allowed here" No quotation marks around text with some of these special characters like a colon: comment: Make sure to read the rules: https... "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '<block mapping start>'" Line indentation is inconsistent "YAML parsing error in section #: while scanning a quoted scalar [...] found unexpected end of stream" Missing a single-quote at the end of an item: body: 'keyword "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '<scalar>'" Separator line (---) is indented, or: A non-doubled apostrophe in an item that's wrapped in single-quotes: 'I'm', or: Missing a double-quote at the end of an item: body: "keyword "YAML parsing error in section #: while scanning a simple key [...] could not found expected ':'" Not enough dashes in a separator line / wrong type of dashes, or: No space after a colon: body:"keyword", or: No colon after a check/etc.: body "keyword", or: Missing an indented dash at the start of a line in a check with a "dictionary"-type list (i.e. a vertical list instead of a horizontal list separated by commas) "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '<block sequence start>'" A line with a different indention than the rest in a check with a "dictionary"-type list (i.e. a vertical list with dashes instead of a horizontal list separated by commas) "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found '-'" A line with no indentation in a check with a "dictionary"-type list (i.e. a vertical list with dashes instead of a horizontal list separated by commas) "YAML parsing error in section #: while parsing a flow sequence [...] expected ',' or ']', but got '<scalar>'" Missing a double-quote at the end of an item:


```yaml
body: ["keyword], or: Missing a single-quote at the end of an item: body: ['keyword1, 'keyword2'], or: Missing a closing bracket: body: ['keyword1', 'keyword2', or: Missing a comma in a list of quoted items bracket: body: ["keyword1" "keyword2"]
```


"YAML parsing error in section #: while parsing a block node expected the node content, but found '?' Too many dashes in a separator line "YAML parsing error in section #: while parsing a block mapping [...] expected <block end>, but found ','" Missing an opening bracket: body: 'keyword1', 'keyword2'] "Can't use combined_karma on this type in rule:" Profile-related check line isn't indented under author "YAML parsing error in section #: while scanning a block scalar [...] expected a comment or a line break, but found '[...]' Non-quoted "bigger than" karma/account age check: comment_karma: > 5, or: First line of a multi-line comment/message/modmail format isn't indented under the function: `comment: "YAML parsing error in section #: unacceptable character [...]: special characters are not allowed" Including characters that are unsupported by Automod's configuration page (like emojis) "Invalid search check: [...] in rule:" No parenthesis for the specified check modes: body includes: "keyword", or: Missing underscore in a function's multi-word name like comment stickied and is top level "invalid value for type: Submission in rule:" Capitalized value in a type check: type: Submission "YAML parsing error in section #: while scanning a double-quoted scalar [...] found unknown escape character" Single-escaping in a double-quoted regex: body (regex): "\w+" "Generated an invalid regex for [...]: multiple repeat at position # in rule:" Double asterisks in a regex: '.**' No error but it won't save Listing multiple types which isn't supported: type: ["link submission", "comment"], or: Including unsupported characters in the "reason for revision" field when saving Return to top

---

## Library of Common Rules

### Introduction and General Notes

If you haven't yet, you'll first need to enable AutoModerator for your subreddit. All you need to do is go to /r/YOURSUBREDDIT/wiki/config/automoderator. The wiki system will ask you if you want to create the page. Say "yes", and you're good to go! Some of these conditions can be directly copy-pasted and some will require filling in of custom details. Some of these rules use modmail to alert moderators. On larger subreddits, you might find it less noisy to remove modmail (and modmail_subject if present) and change the rule to use action: filter. That has the effect of temporarily removing the submission or comment until a moderator can review it from the moderation queue. When copying rules into your AutoModerator, --- separator lines must not be indented. While AutoModerator is flexible about line indentation as long as it is consistent, it is recommended that top-level lines be indented with 4 spaces and second-level lines with an additional 4 spaces. Try to resist the temptation to add rules before there's evidence they're needed!

### Commenting

on Submissions --- # Sticky comment on submissions


```yaml
type: submission
is_edited: false
comment_stickied: true
comment: |
    The text of the comment goes here.
    Put the same number of spaces at the beginning of each comment line.
    If you want separate paragraphs, keep the empty lines between each paragraph (AutoModerator uses [Markdown](https://www.reddit.com/wiki/markdown#wiki_quick_reference) formatting).
```


--- Automatic Sticky Replies Based on Flair This rule only applies if the post is submitted with one of the listed flairs at the time of submission. AutoModerator does not reevaluate rules if the flair changes later. You can change flair_text to ~flair_text if you want to reverse the flair check and only reply to submissions that are not flaired with one of the listed flairs. --- # Sticky comment on submissions


```yaml
type: submission
is_edited: false
```


# Don't act again if the post is edited


```yaml
flair_text (includes-word): ["Flair1", "Flair2"]
comment_stickied: true
comment: |
    The text of the comment goes here (more explanation in the previous rule).
```


---

### Content Control

These rules help restrict what can be posted.

Domain Blacklist Prevent websites from being linked to or mentioned anywhere on your subreddit. Useful for banning phishing, spamming, or other malicious sites. The use of action: spam removes the content and trains your subreddit spam filter to catch similar sites in the future. If you want to see these posts in your modqueue so you can ban these accounts, change action: spam to action: filter. If the Domain Whitelist rule is also being used, remove domain+ from the first line of the rule because the Domain Whitelist rule handles submissions with a domain more broadly. If you use a Domain Blacklist, it is a good idea to also use the URL Shortener Blacklist to prevent users from circumventing the blacklist with URL shorteners. Replace the list between the brackets with your own list of sites to ban.


```yaml
---
domain+body+title: [badwebsite1.com, badwebsite2.com, badwebsite3.com]
action: spam
action_reason: "Spam domain [{{match}}]"
---
    Subreddits with lots of youtube or other video submissions can add +media_description to also enforce the blacklist in the description of submitted videos:
---
domain+body+title+media_description: [badwebsite1.com, badwebsite2.com, badwebsite3.com]
action: spam
action_reason: "Spam domain [{{match}}]"
---
    Domain Whitelist Only allow submissions to certain websites. If you are also using the Domain Blacklist rule, remove the domain+ from the first line of that rule because this rule handles submissions with a domain more broadly (i.e., removes more domains). Replace the list between the brackets with your own list of approved domains. If you want to allow self posts (text posts), you can either add a domain of self.subredditname (replace subredditname with the name of your subreddit) or add the line
type: link submission
```


.


```yaml
---
~domain: [site1.com, site2.com, site3.com]
action: remove
action_reason: "Unapproved domain [{{domain}}]"
comment: |
    Your submission was automatically removed because {{domain}} is not an approved site.
```


--- Domain Un-Spam List This approves submissions to known safe sites so they won't be caught in the spam filter:


```yaml
---
domain: [site1.com, site2.com, site3.com]
action: approve
action_reason: "Unspam domain [{{match}}]"
---
    Subreddit Blacklist Prevent certain subreddits from being linked to anywhere on your subreddit. Replace the subreddit lists with your own list of subreddits to ban. Change either or both rules from remove to filter if you want these submissions to be added to the moderation queue to allow a moderator to review the post or the account making the post. The third removal rule is to prevent the use of short link formats to circumvent the subreddit blacklist.
---
domain+url+body: [r/badsubreddit1, r/badsubreddit2]
message: |
    Your submission was automatically removed because that is a disallowed link subreddit.
action: remove
action_reason: "Bad subreddit [{{match}}]"
---
# Remove crossposts from disallowed subreddits
type: crosspost submission
crosspost_subreddit: name: [bannedsubreddit1, bannedsubreddit2]
message: |
    Your submission was automatically removed because that is a disallowed crosspost subreddit.
action: remove
action_reason: "Crosspost from a disallowed subreddit"
---
domain+url+body (regex): ['redd\.it/\w+', 'reddit\.com/comments/\w+']
action: remove
action_reason: "Reddit short link [{{match}}]"
message: |
    Your {{kind}} in /r/{{subreddit}}) was automatically removed.
    Short links are not permitted in /r/{{subreddit}} as they impair our ability to enforce link blacklists.
```


--- Crosspost Whitelist These rules will remove crossposts and link posts to subreddits that are not whitelisted. --- # Only allow crossposts from whitelisted subreddits type: crosspost submission


```yaml
crosspost_subreddit: ~name: [approvedsubreddit1, approvedsubreddit2]
message: |
    Your submission was automatically removed because that is not an approved crosspost subreddit.
action: remove
action_reason: "Crosspost from unapproved subreddit"
---
# Only allow link posts to whitelisted subreddits
type: link submission
url (regex): ['reddit\.com/r/(?!approvedsubreddit1|approvedsubreddit2)']
message: |
    Your submission was automatically removed because that is not an approved link subreddit.
action: remove
action_reason: "Link post to an unapproved subreddit"
---
    Uploaded media detection in comments This regex can be used to detect images or gifs uploaded in comments Using regex in automod to detect media in comments: !\[(?:gif|img)\]\(([^\|\)]+(?:|\|[^\|\)]+))\) Adding automoderator comment to media in comments:
type: comment
body (includes, regex): ['!\[(img|gif)\]\(((?!emote|static_png|giphy)[-\w\|]+)\)']
comment: "this is an image upload (and not an emote/giphy)!"
```


Using a filter for uploaded media in comments:


```yaml
type: comment
body (regex, includes): ['!\[(?:gif|img)\]\(([^\|\)]+(?:|\|[^\|\)]+))\)']
action: filter
action_reason: "Media in comments"
```


Non-English Content Ban These may be copied and pasted to your AutoModerator configuration without modification. If donger is common on your subreddit, the first rule is not recommended. The first rule matches about 75% to 80% of non-English spam on Reddit. The latter rules will not predict the language accurately, they are only intended to identify non-English content. If you are experienced with AutoModerator and regex, the first rule may be adapted for use on non-English subreddits: Remove any ranges in the first rule that may match on a language you want to allow. Note that many languages based on the Latin alphabet use characters from one or more Latin Extended Unicode ranges. Many languages also use the Combining Diacritical Marks Unicode range. The additional rules are not recommended for use on non-English subreddits. On English-only subreddits experiencing severe problems with non-English content, you may consider adding "(?#Latin-1 Supplement)(?-i:[ÀÁÃÄÆÇÈÊÍÎÐÑÓÖØÚÛÜÝÞßàãæëìîðòõøùúýþ]+)" to the first rule. These rules are still under development and may be buggy. Use at your own risk.


```yaml
---
body+title (regex, includes): ["(?#Latin Extended-A)(?-i:[\u0100-\u017f]+)", "(?#Latin Extended-B)[\u0180-\u024f]+", "(?#Combining Diacritical Marks)[\u0300-\u0335\u0337-\u0360\u0362-\u036f]+", "(?#Cyrillic)[\u0400-\u052f]+", "(?#Hebrew)[\u0590-\u05ff]+", "(?#Arabic)[\u0600-\u0669\u066b-\u06ff]+", "(?#Devanagari)[\u0900-\u097f]+", "(?#Bengali)[\u0980-\u09ff]+", "(?#Gurmukhi)[\u0a00-\u0a7f]+", "(?#Tamil)[\u0b80-\u0bff]+", "(?#Kannada)[\u0c80-\u0cff]+", "(?#Thai)[\u0e00-\u0e7f]+", "(?#Latin Extended Additional)[\u1e00-\u1eff]+", "(?#Hiragana)[\u3041-\u3096]+", "(?#Katakana)[\u30a1-\u30c3\u30c5-\u30fa]+", "(?#CJK Unified Ideographs)[\u4e00-\u9fff]+", "(?#Hangul)[\uac00-\ud7af]+"]
action: filter
action_reason: "Non-English [{{match}}]"
---
type: submission
title+body (regex, includes): ['(?-i:[ÀÂÆÇÈÉÊËÎÏÔÙÛÜàâæçèêëîïôùûüÿŒœŸ])']
body+title (regex): ['(?<!\bdu.)jours?', '(?<!\blaissez.)faire', 'a(insi|lors|ucune|ujourd[\x27’]hui|ussi|utres?|vait|vec|voir)', 'b(iens?|onnes?)', 'c([\x27’]est(?!.magnifique\b)|ela|es|ette|hez|omme|omptes?|ontre)', 'd([\x27’]autres|[\x27’]un|[\x27’]une|ans|epuis|eux|its?|onc|roite)', 'e(ffets?|lles?|ntre|ntreprises?)', 'f(aits?|aut|ois)', 'gauche', 'ils', 'j([\x27’]ai|amais)', 'l([\x27’]on|es|eurs?|oi|ui)', 'm(ais|oi|oins|ois|onde)', 'n([\x27’]a|[\x27’]est|iveau|ous|ouveau|ouvelles?)', 'oui', 'p(artie|as|ersonnes?|eu|eut|euvent|eux|ourquoi|roduits?|utain)', 'qu([\x27’]ils?|[\x27’]on|and|elles?|elques|els?|i|oi)', 'r(este|ien)', 's([\x27’]est|elon|erait|oit|ont|ouvent|uis|ur)', 't([\x27’]as|[\x27’]es|ermes?|itres?|oujours?|ous|outes?|rois|rop|rouves?)', 'une', 'v(ais|ers|oir(?!.dire\b)|ous)']
action: filter
action_reason: "Non-English (French) [{{match-title+body}}], [{{match-body+title}}]"
---
type: submission
title+body (regex, includes): ['[ÄÖÜßäöü]']
body+title: ['(aber|alles|als|auch|auf|bei|bist|bitte|damit|danke|dann|dass|dein|deine|dem|denn|der|des|diese|dieser|dir|doch|ein|eine|einem|einen|einer|einfach|etwas|euch|frau|ganz|gehen|geht|gesagt|gibt|gott|hab|haben|hast|hatte|heute|hier|ihm|ihn|ihnen|ihr|immer|jetzt|kann|kannst|kein|keine|komm|kommen|kommt|leben|leute|los|machen|mehr|meine|meinen|mich|mit|nein|nicht|nichts|nie|noch|nur|oder|sagen|schon|sehen|sehr|sein|sich|sicher|soll|und|uns|viel|von|vor|warum|wenn|werde|werden|wie|wieder|willst|wirklich|wissen|wollen|wollte|wurde|zeit|zum|zur)']
action: filter
action_reason: "Non-English (German) [{{match-title+body}}], [{{match-body+title}}]"
---
type: submission
title+body (regex, includes): ['(?-i:[¡ªº¿ÀÁÂÃÇÈÉÊÌÍÑÒÓÔÕÙÚÜàáâãçèêìíñòóôõùúü])']
body+title (regex): ['a(lgo|cha|cho|hora|inda|lguém|lguien|nos|penas|qui|ssim|té)', 'b(em|ueno)', 'c(asa|erto|oisa|oisas|omo|osa|osas|reo|uando)', 'd(ecir|epois|esde|espués|eus|eve|ia|ije|ijo|ios|isse|izer|ois|onde)', 'e(la|le|les|llos|ntonces|res|sa|se|so|spera|ssa|sse|sta|staba|stamos|star|stas|stava|ste|sto|stou|stoy)', 'f(alar|az|azendo|azer|icar|oi|ue|uera)', 'g(ente|racias)', 'h(ablar|ace|acer|echo|ijo|ola|ombre|omem)', 'i(sso|sto)', 'l(he|ugar)', 'm(ais|ejor|elhor|esmo|eu|ierda|inha|is|ismo|omento|ucho|uito|undo|uy)', 'n(ada|adie|em|oche|oite|os|osotros|unca)', 'o(brigado|nde|tra|tro)', 'p(ai|arece|asa|elo|ero|essoas|ode|odemos|or|orque|osso|ouco|ra|reciso|uede|uedes|uedo)', 'qu(al|ando|é|em|er|ero|ién|iere|ieres|iero)', 's(abes|eguro|ei|em|empre|enhor|eu|eus|iempre|iento|obre|ua)', 't(alvez|ambém|ambién|em|emos|enemos|engo|enho|er|iempo|iene|ienes|inha|ipo|odo|odos|rabajo|udo|us)', 'u(ma|no|sted)', 'v(amos|er|erdad|erdade|ez|ida|ou|oy)']
action: filter
action_reason: "Non-English (Spanish or Portuguese) [{{match-title+body}}], [{{match-body+title}}]"
---
    Spam Obfuscations Spammers often try to obfuscate text by replacing letters with similar-looking characters from various Unicode regions. This rule will also match on some non-English languages. On English-only subreddits experiencing severe problems with spam obfuscations, you may consider adding "(?#Latin-1 Supplement)(?-i:[ÀÁÃÄÆÇÈÊÍÎÐÑÓÖØÚÛÜÝÞßàãæëìîðòõøùúýþ]+)" to this rule. This rule is still under development and may be buggy. Use at your own risk.
---
body+title (regex, includes): ["(?#Latin Extended-A)(?-i:[\u0100-\u017f]+)", "(?#Latin Extended-B)[\u0180-\u024f]+", "(?#IPA Extensions)[\u0250-\u02af]+", "(?#Spacing Modifier Letters)[\u02b0-\u02ff]+", "(?#Combining Diacritical Marks)[\u0300-\u0335\u0337-\u0360\u0362-\u036f]+", "(?#Greek and Coptic)[\u0370-\u03ff]+", "(?#Cyrillic)[\u0400-\u052f]+", "(?#Armenian)[\u0530-\u058f]+", "(?#Cherokee)[\u13a0-\u13ff]+", "(?#Unified Canadian Aboriginal Syllabics)[\u1400-\u167f]+", "(?#Phonetic Extensions)[\u1d00-\u1d7f]+", "(?#Phonetic Extensions Supplement)[\u1d80-\u1dbf]+", "(?#Latin Extended Additional)[\u1e00-\u1eff]+", "(?#Greek Extended)[\u1f00-\u1fff]+", "(?#Letterlike Symbols)(?-i:[\u2100-\u214f]+)", "(?#Number Forms)[\u2160-\u218b]+", "(?#Enclosed Alphanumerics)[\u2460-\u24ff]+", "(?#Glagolitic)[\u2c00-\u2c5f]+", "(?#Latin Extended-C)[\u2c60-\u2c7f]+", "(?#Coptic)[\u2c80-\u2cff]+", "(?#Latin Extended-D)[\ua720-\ua7ff]+", "(?#Latin Extended-E)[\uab30-\uab6f]+", "(?#Cherokee Supplement)[\uab70-\uabbf]+", "(?#Halfwidth and Fullwidth Forms)[\uff00-\uff0c\uff0e-\uffef]+", "(?#Mathematical Alphanumeric Symbols)[\U0001D400-\U0001D7FF]+", "(?#Enclosed Alphanumeric Supplement)[\U0001F100-\U0001F1FF]+"]
action: filter
action_reason: "Possible spam obfuscations or non-English [{{match}}]"
---
    URL Shortener Ban If you use the Domain Blacklist rule, we recommend also using this rule to prevent blacklist circumvention.
---
domain+body+title: ['http://redd.it', 'https://redd.it', 0rz.tw, 1jl.info, 1link.in, 1un.fr, 1url.com, 1url.cz, 1wb2.net, 2.gp, 2.ht, 2big.at, 2doc.net, 2fear.com, 2pl.us, 2tu.us, 2u.xf.cz, 2ya.com, 3.ly, 3x.si, 4ms.me, 4sq.com, 5z8.info, 6g6.eu, 7.ly, 7li.in, 8u.cz, a.co, a.gg, a.nf, a0.fr, a2n.eu, aa.cx, abbrr.com, abnb.me, ad-med.cz, ad.vu, ad5.eu, ad7.biz, adb.ug, adcraft.co, adcrun.ch, adf.ly, adfa.st, adflav.com, adjix.com, adv.li, afx.cc, aka.gr, alturl.com, amzn.to, any.gs, app.link, app.x.co, apple.news, ar.gy, asso.in, atu.ca, azc.cc, b23.ru, b2l.me, b54.in, b65.us, bc.vc, bcool.bz, beam.to, bee4.biz, bfy.tw, bigly.us, bim.im, binged.it, bit.do, bit.ly, bitly.com, bitw.in, bizj.us, bkite.com, bl.ink, blap.net, ble.pl, blip.tv, bote.me, bougn.at, bravo.ly, brk.to, brzu.net, bsa.ly, bst.is, budurl.com, buff.ly, burnurl.com, bv.ms, bxl.me, bzh.me, canurl.com, cbug.cc, cc.cc, cektkp.com, cf2.me, cf6.co, chilp.it, chzb.gr, cjb.net, cl.ly, clck.ru, cli.gs, cli.re, cliccami.info, clickmeter.com, clickthru.ca, clikk.in, clk.im, clnk.in, cnn.it, conta.cc, cort.as, cot.ag, crisco.com, crks.me, crwd.cr, ctvr.us, cur.lv, cutt.eu, cutt.ly, cutt.us, cuturl.com, cybr.fr, cyonix.to, dai.ly, db.tt, dd.ma, decenturl.com, dfl8.me, dft.ba, digbig.com, digg.com, disq.us, dld.bz, dlvr.it, do.my, dopice.sk, doshort.com, droid.ws, dwarfurl.com, dy.fi, dyo.gs, easyurl.com, easyurl.net, ebay.to, ecra.se, eepurl.com, erw.cz, esyurl.com, eweri.com, exe.io, ezurl.cc, fa.by, fav.me, fb.me, fbshare.me, ff.im, fhurl.com, filoops.info, filz.fr, fire.to, firsturl.de, firsturl.net, fivr.me, flic.kr, flq.us, fly2.ws, freze.it, fur.ly, fvrr.co, fwd4.me, fwib.net, g.co, g00.me, geniuslink.com, get-shorty.com, gg.gg, gizmo.do, go.9nl.com, go.ign.com, go.usa.gov, go2.me, go2cut.com, golinks.co, goo.gl, goshrink.com, gowat.ch, gurl.es, hellotxt.com, hex.io, hide.my, hiderefer.com, hit.my, hmm.ph, hops.me, hover.com, href.li, hsblinks.com, ht.ly, htxt.it, hubs.ly, huff.to, hurl.it, hyperurl.co, icit.fr, ick.li, icks.ro, idek.net, ift.tt, iguang.tw, iiiii.in, iky.fr, ilix.in, is.gd, iscool.net, itm.im, ity.im, ix.lt, ix.sk, j.gs, j.mp, jdem.cz, jmp2.net, jqw.de, just.as, kask.us, kd2.org, kfd.pl, kissa.be, korta.nu, kr3w.de, kratsi.cz, krod.cz, krunchd.com, kuc.cz, l-k.be, l9.fr, l9k.net, labb.in, lat.ms, lc-s.co, lc.cx, lemde.fr, libero.it, liip.to, liltext.com, linkbun.ch, linkto.im, linktr.ee, linx.cf, llu.ch, lnk.co, lnk.ms, lnk.sk, lnkd.in, lnks.fr, lru.jp, lt.tl, m3mi.com, macte.ch, mailchi.mp, mcaf.ee, mdl29.net, merky.de, metamark.net, mic.fr, migre.me, mke.me, mktw.net, moby.to, mol.im, moourl.com, more.sh, mrte.ch, myurl.in, mz.cm, n.pr, nanoref.com, nbc.co, nblo.gs, net46.net, nicou.ch, nig.gr, not.my, notlong.com, nov.io, nq.st, nsfw.in, nutshellurl.com, nyti.ms, o-x.fr, oc1.us, okok.fr, on.mktw.net, onelink.me, onforb.es, oua.be, ow.ly, oze.io, p6l.org, parky.tv, past.is, ph.ly, picz.us, pin.st, ping.fm, plots.fr, pm.wu.cz, po.st, politi.co, poprl.com, post.ly, posted.at, ppt.cc, ppt.li, prettylinkpro.com, ptiturl.com, ptm.ro, pub.vitrue.com, q.gs, qbn.ru, qicute.com, qlnk.net, qqc.co, qqurl.com, qr.ae, qr.net, qrtag.fr, qte.me, quip-art.com, qxp.sk, qy.fi, r.im, rb.gy, rb6.me, read.bi, readthis.ca, redirects.ca, redirx.com, redu.it, ref.so, relink.fr, reut.rs, rite.link, rsmonkey.com, rt.nu, rurl.org, rx.hu, s-url.fr, safe.mn, sagyap.tk, scrnch.me, sdu.sk, sdut.us, sh.st, shar.as, shar.es, sharein.com, sharetabs.com, shink.de, shor.by, shorl.com, short.cm, short.pk, short.to, shorte.st, shorten.me, shortenurl.com, shorterlink.com, shortn.me, shortna.me, shorturl.at, shorturl.com, show.my, shredurl.com, shrinke.me, shrinkify.com, shrinkr.com, shrinkurl.us, shrt.fr, shrt.in, shrten.com, shrtnd.com, shurl.net, sicax.net, simurl.com, sina.lt, skroc.pl, slate.me, smallr.com, smarturl.it, smsh.me, snip.ly, snipr.com, snipurl.com, snsw.us, snurl.com, soo.gd, sq6.ru, sqrl.it, srnk.net, starturl.com, sturly.com, surl.co.uk, surl.me, sy.pe, t.cn, t.co, t.lh.com, t.me, t2m.io, tabzi.com, tcrn.ch, tdjt.cz, tgr.ph, thn.li, tighturl.com, tiks.co, tin.li, tiny.cc, tiny.lt, tiny.pl, tiny.tw, tinyarrows.com, tinylink.com, tinylink.in, tinyurl.com, tinyurl.hu, tl.gd, tldr.sk, tmi.me, tnw.to, tny.com, tny.cz, to.ly, to8.cc, togoto.us, tohle.de, tpmr.com, tprt.co, tr.im, tr5.in, tra.kz, traceurl.com, trck.me, trunc.it, tweetburner.com, tweez.me, twet.fr, twhub.com, twirl.at, twitclicks.com, twitterpan.com, twiturl.de, twurl.cc, twurl.nl, tyn.ee, u.mavrev.com, u.nu, u.to, u6e.de, ug.cz, ukl.me.uk, upzat.com, ur1.ca, url.ie, url.lotpatrol.com, url4.eu, url4u.co, url5.org, urladda.com, urlao.com, urlborg.com, urlcut.com, urlcutter.com, urlhawk.com, urlin.it, urlpire.com, urls.fr, urls.tn, urltea.com, urlx.ie, urlz.fr, usat.ly, utfg.sk, v.gd, v.ht, vaza.me, vbly.us, vd55.com, verd.in, vgn.am, vgn.me, viralurl.biz, viralurl.com, virl.ws, vm.lc, vov.li, vrl.to, vt802.us, vur.me, vurl.bz, vurl.com, vzturl.com, w1p.fr, w55.de, wa.link, wa.me, waa.ai, wapo.st, wb1.eu, web99.eu, wed.li, win.gy, work.ink, workink.co, wp.me, wu.cz, ww7.fr, x.co, x.vu, x2c.eu, xaddr.com, xav.cc, xil.in, xl8.eu, xoe.cz, xr.com, xrl.in, xrl.us, xtu.me, xurl.es, yatuc.com, yeca.eu, yfrog.com, yhoo.it, yiyd.com, yogh.me, youfap.me, yourls.org, yourname.shim.net, ysear.ch, yuarel.com, yweb.com, yyv.co, z0p.de, z9.fr, zapit.nu, zeek.ir, zi.ma, zi.mu, zi.pe, zip.net, zud.me, zurl.ws, zxq.net, zz.gd, zzb.bz]
action: remove
action_reason: "URL shortener [{{match}}]"
message: "Your {{kind}} has been removed because you used a URL shortener ({{match}}). Please only use direct and full-length URLs."
---
    Crowdfunding This rule can be copy-pasted directly to your AutoModerator configuration without modification.
---
body+domain+title+url: [begslist.com, booster.com, cash.app, cash.me, charityvest.org, crowdfunder.co.uk, crowdrise.com, donorschoose.org, firstgiving.com, fnd.us, fundanything.com, fundly.com, fundrazr.com, generosity.com, gf.me, gfwd.at, givealittle.co.nz, giveforward.com, givesendgo.com, gofund.me, gofundme.com, goget.fund, gogetfunding.com, igg.me, indiegogo.com, justgiving.com, kck.st, ketto.org, kickbooster.me, kckb.st, kickstarter.com, launchfinance.com.au, m-lp.co, patreon.com, payfriendz.me, payit2.com, payitsquare.com, paypal.com/cgi-bin, paypal.com/paypalme, paypal.me, petcaring.com, pitchfuse.com, redditmade.com, sponsorchange.org, tilt.com, tilt.tc, totalgiving.co.uk, youcaring.com, youcaring.net, youcaring.org]
action: filter
action_reason: "Crowdfunding [{{match}}]"
---
    Petitions
---
body+title: [act.rootsaction.org, actblue.com, action.sumofus.org, activism.thenation.com, avaaz.org, change.org, chn.ge, chng.it, credomobilize.com, demandprogress.org, easypolls.net, go.berniesanders.com, gopetition.com, leftaction.com, moveon.org, petitions.whitehouse.gov, signon.org, startjoin.com, strawpoll.me, takepart.com, thepetitionsite.com, watchdog.net]
action: remove
action_reason: "Petition [{{match}}]"
comment: "Your {{kind}} has been removed. Petitions aren't allowed here."
---
    Surveys and Polls
---
body+title+url (regex): ['(dashpoll|midzy|qualtrics|typeform)\.com', '[\w.-]*(strawpoll|survey)[\w.-]*\.(com?|me|uk)(\.[\w-]+)*', 'crowdsignal\.com', 'docs\.google\.com/(a/[^/]+/)?forms(?=/)', 'forms\.gle', 'instant\.ly', 'reddit\.com/poll', 'survey\.fm', 'survey\.zohopublic\.com', 'survio\.com', 'wufoo\.com/forms']
action: remove
action_reason: "Survey link [{{match}}]"
comment: "Your {{kind}} has been removed. Surveys and polls aren't allowed here."
---
    Require Direct Image Links This is a set of several rules which work together to remove image-hosting links that do not play nicely with RES and mobile apps, and, where possible, generates a direct link for the user to re-submit. Two of the rules are for imgur, which has several different types of URL formats. This set of five rules can be copy-pasted directly to your AutoModerator configuration without modification.
---
domain: [gyazo.com, ibb.co, imageshack.us, pinterest.com, postimg.cc, postimg.org, prnt.sc, prntscr.com, puu.sh]
~url (ends-with): [.gif, .jpeg, .jpg, .png]
action: remove
action_reason: "Indirect link to hosted image #1 [{{url}}]"
comment: |
    Your submission has been automatically removed due to an indirect link to a hosted image.
    Please be considerate of mobile and RES users, and resubmit your link with the proper file extension.
    For your convenience, here is your submitted link with a .png file extension appended to the end.
    Please check that it works.
    If it does, retry your submission with this link: {{url}}.png If the above link does not work, right-click on your image, select *Copy Image URL*, and paste that into the reddit submission page instead.
---
domain: [imgur.com]
url (regex): ['imgur\.com/(a/)?[A-Za-z0-9]{5,8}$']
action: remove
action_reason: "Indirect link to hosted image #2 [{{url}}]"
comment: |
    Your submission has been automatically removed due to an indirect link to a hosted image.
    Please be considerate of mobile and RES users, and resubmit your post as a direct link.
    For your convenience, here is your submitted link with .jpg and .gif file extensions appended to the end.
    Please see if one works, and re-try your submission with one of the following links: * {{url}}.jpg * {{url}}.gif If the above links do not work, right-click on your image, select *Copy Image URL*, and paste that into the reddit submission page.
---
domain: [imgur.com]
url (regex): ['gallery', 'imgur\.com/[A-Za-z0-9]{5,8},([A-Za-z0-9]{5,8},?)+']
action: remove
action_reason: "Indirect link to hosted image #3 [{{url}}]"
comment: |
    Your submission has been automatically removed due to an indirect link to a hosted image.
    Non-album multi-image links and imgur gallery links are not compatible with RES and mobile apps.
    For multi-image links, please create an album and submit a link to that.
    For gallery images, please right-click your image, choose *Copy Image URL*, and submit that direct image link to reddit.
---
domain: [imgflip.com, pinterest.com, snag.gy, uput.in]
~url: [i.imgflip.com, i.snag.gy, i.uput.in]
action: remove
action_reason: "Indirect link to hosted image #4 [{{url}}]"
comment: |
    Your submission has been automatically removed due to an indirect link to a hosted image.
    Please right-click your image, choose *Copy Image URL*, and submit that direct image link to reddit.
---
domain: [tinypic.com]
action: remove
action_reason: "Indirect link to hosted image #5 [{{url}}]"
comment: |
    Your submission has been automatically removed due to an indirect link to a hosted image.
    Unfortunately, the media host {{domain}} is not compatible with mobile apps and/or RES.
    Please be considerate of mobile and RES users and resubmit your content using a different media host.
```


--- Profanity Filter This filters variants of common vulgarity including semi-censored variants and some common Spanish profanity. This rule can be copy-pasted directly to your AutoModerator configuration without modification, but some fine-tuning may be required.


```yaml
---
title+body (regex): ['((bul+|dip|horse|jack).?)?sh(\\?\*|[ai]|(?!(eets?|iites?)\b)[ei]{2,})(\\?\*|t)e?(bag|dick|head|load|lord|post|stain|ter|ting|ty)?s?', '((dumb|jack|smart|wise).?)?a(rse|ss)(.?(clown|fuck|hat|hole|munch|sex|tard|tastic|wipe))?(e?s)?', '(?!(?-i:Cockburns?\b))cock(?!amamie|apoo|atiel|atoo|ed\b|er\b|erels?\b|eyed|iness|les|ney|pit|rell|roach|sure|tail|ups?\b|y\b)\w[\w-]*', '(?#ES)(cabr[oó]n(e?s)?|chinga\W?(te)?|g[uü]ey|mierda|no mames|pendejos?|pinche|put[ao]s?)', '(?<!\b(moby|tom,) )(?!(?-i:Dick [A-Z][a-z]+\b))dick(?!\W?(and jane|cavett|cheney|dastardly|grayson|s?\W? sporting good|tracy))s?', '(cock|dick|penis|prick)\W?(bag|head|hole|ish|less|suck|wad|weed|wheel)\w*', '(f(?!g\b|gts\b)|ph)[\x40a]?h?g(?!\W(and a pint|ash|break|butt|end|packet|paper|smok\w*)s?\b)g?h?([0aeiou]?tt?)?(ed|in[\Wg]?|r?y)?s?', '(m[oua]th(a|er).?)?f(?!uch|uku)(\\?\*|u|oo)+(\\?\*|[ckq])+\w*', '[ck]um(?!.laude)(.?shot)?(m?ing|s)?', 'b(\\?\*|i)(\\?\*|[ao])?(\\?\*|t)(\\?\*|c)(\\?\*|h)(e[ds]|ing|y)?', 'c+u+n+t+([sy]|ing)?', 'cock(?!-ups?\b|\W(a\Whoop|a\Wsnook|and\Wbull|eyed|in\Wthe\Whenhouse|of\Wthe\W(rock|roost|walk))\b)s?', 'd[o0]+u[cs]he?\W?(bag|n[0o]zzle|y)s?', 'piss(ed(?! off)(?<!\bi(\sa|\W?)m pissed)|er?s|ing)?', 'pricks?', 'tit(t(ie|y))?s?']
action: filter
action_reason: "Profanity [{{match}}]"
---
    After a sufficient period of testing, you may consider replacing the latter two lines with the following:
---
action: remove
action_reason: "Profanity [{{match}}]"
message: |
    Your [{{kind}}]({{permalink}}) in /r/{{subreddit}} was automatically removed. /r/{{subreddit}} is geared towards younger users so please watch your language.
```


--- You can optionally change the message to better suit your particular subreddit. Match Multiple Keywords To match multiple keywords from separate lists being checked on the same field(s), add a # name to the check. Without the # name, each subsequent check of the same type will overwrite the previous check of that type. Do not use a number as the # name because that can break {{match}} placeholders in some situations. --- title+body#color: ['blue', 'green', 'red'] title+body#shape (regex): ['circles?', 'squares?', 'triangles?']


```yaml
action: filter
action_reason: "Colored shape filter [{{match-title+body#color}}] [{{match-title+body#shape}}]"
---
    This can also be done with regular expressions:
---
title+body (regex): ['\b(blue|green|red)\b.*?\b(circle|square|triangle)s?\b', '\b(circle|square|triangle)s?\b.*?\b(blue|green|red)\b']
action: filter
action_reason: "Colored shape filter [{{match}}]"
---
    Another option is using positive-lookahead in a regex, but this is only recommended if you are experienced with more complex regular expressions:
---
title+body (regex): ['^(?=.*?\b(blue|green|red)\b).*?\b(circle|square|triangle)s?\b']
action: filter
action_reason: "Colored shape filter [{{match}}]"
```


---

### Content Quality Control

These rules focus on low-effort or low-quality content.

low-effort or low-quality content (e.g., "shitposts"). Common Clickbait Titles This rule flags posts which may be "clickbait" (low-effort articles intended to generate clicks and ad revenue rather than provide meaningful information). The original version of this rule was based on this submission from /r/dataisbeautiful.


```yaml
---
title+media_title (regex): ['(10|\d+\b(?<!covid.19)|five|four|one|seven|simple|six|three|two) ((\w+ )?(?-i:Ways)|easy|best|free|main|money|reasons?|steps)', '([5-9]|\d\d+|five|seven|simple|six) (\w+ )?ways', '(\d+|five|four|one|only|pro|seven|simple|six|this|three|two|weird)(\W[\w\x27-]{3,})?\W((pro\W?)?tips|things? (every\w*|one|only|you\w*)|trick)s?', '(photos|pictures|images) that prove', '\d{1,2} (signs|reasons) (you(\W?re)?|why)', '\d{1,2} dogs who', '\d{1,2} most important', '\d{1,2} things that', 'are the most', 'before you die', 'blow your mind', 'character are you', 'd(id|o)n\W?t know about', 'game of thrones', 'in real life', 'in your life', 'is this the', 'probably d(id|o)n\W?t know', 'reasons you should', 'things that (actually |really )?happen(ed)?', 'things you d(id|o)n\W?t', 'things you probably', 'will blow your', 'you probably d(id|o)n\W?t', 'you should be']
action: report
action_reason: "Possible clickbait [{{match}}]"
---
    Short Top-Level Comments Comments below a certain length are unlikely to contribute to discussion. Depending on the needs of your subreddit, you may want to remove such comments. This rule removes top-level comments that are less than 11 characters. This rule can be copy-pasted directly to your AutoModerator configuration without modification. Optionally, you can adjust the required comment length. If you want to remove all comments below the threshold, rather than just top-level comments, remove the is_top_level line. If you want this to apply to text-posts as well, remove the type line.
---
type: comment
body_shorter_than: 11
is_top_level: true
action: remove
action_reason: "Short top-level comment"
---
    Link-only Self Posts This rule can be copy-pasted directly to your AutoModerator configuration without modification.
---
type: text submission
body (regex, full-text): ['(\[[^\]]*\]\()?https?://\S+\)?']
action: remove
action_reason: "Link-only self post"
---
    Self Posts without Text This rule can be copy-pasted directly to your AutoModerator configuration without modification.
---
type: text submission
body_shorter_than: 1
action: remove
action_reason: "Self post without text"
---
    Walls of Text This rule sends a message in response to submissions and comments that have really long paragraphs because they may be difficult to read. This rule can be copy-pasted directly to your AutoModerator configuration without modification. Some subreddits may want to add an
action: filter
    and action_reason line to allow moderators to review matching submissions and comments.
---
body (regex, includes): ['[^\n]{2000}', '^\W*[^\n]{1750,}\W*$']
message: "Please add some paragraph breaks to [your {{kind}}]({{permalink}}) by placing a blank line between distinct sections."
---
    Comments that are only "MRW" or "MFW" links
---
type: comment
body (regex, full-text): ['\[M[RF]W\]\(https?://\S+\)']
action: remove
action_reason: "Comment that is only MRW/MFW and a link"
---
    Mobile Links
---
domain (starts-with): [m., mobile.]
action: remove
action_reason: "Mobile link [{{domain}}]"
comment: |
    Your submission was automatically removed because you linked to the mobile version of a website.
    Please submit a non-mobile link instead.
---
    Comments that are just /r links
---
type: comment
body (regex, full-text): ['/?r/\w+']
action: remove
action_reason: "Comment is just /r link [{{match}}]"
---
    Short and Senseless Memes Removes "circlejerky" comments. This rule is a little aggressive and may not be suitable for "less than serious" subreddits. Use at your own risk. For more serious subreddits, it might be acceptable to increase the body_shorter_than to 60 characters. This can be copy/pasted directly to AutoModerator configuration without modification. For smaller subreddits, you could replace
action: remove
```


with


```yaml
action: report or
action: filter.
---
type: comment
body (regex): ['((nick)?name|account)( \S+){0,2}\W+check\w*\W?out', '(3|th?ree)\b.{0,10}\bfi(dd?|ft)y', '(?!carl\b)c[ao]+r+l+', '(?!passwords?\b)p[4a][$5s]{2}\W?w[0o]rd\w*', '(?-i:BOFA)', '(?<!.{32})(?<!( car| job|aged|gher|lea[nr]|nior|uilt|vage) )titles?(?!.*\b(car|company|fee|insurance|job|loan|vehicle)s?\b)', '(?<!\.)(?<!rule of )69(?!( (days|years)|[,.]))', '(?<!thank )(yo)?u sir', '(?<=\$)750', '(\d+|three|too|two)\W?(deep|edge?|meta|spice?|spook|spoop)y?\W?(\d+|for)\W?me', '(\w{3,}|u) had one job', '(^\S*|chop|cut)\w* ?onions(?!,)', '(and|or)\W+my\W+axe?', '((bul+|dip|horse|jack).?)?sh(\\?\*|[ai]|(?!(eets?|iites?)\b)[ei]{2,})(\\?\*|t)e?(s|ty)?', '(chad|karen|nancy)s?', '(comment|post|story|this) (checks out|gave me (\w+\W+){0,2}(aids|autism|cancer)|kills the|will be downvoted)', '(deleted|removed) comments?', '(dia-?)?beet-?us', '(dick|penis) (stuck|in crazy)', '(down|up)\W?(boat|doot|vot)\w*', '(do|lik) dis', '(enter|join)ed (the )?chat\w*', '(gentleman|scholar) and a? ?(gentleman|scholar)', '(i|u|you) (just )?lost the game', '(mis)?read th(at|is) as', '(nope[,.]? ?){2,}', '(tb[fh]|tho)(?=\W*$)', '(top.?)?kek', '(yo(u|u are|u\Wre|ur(?! ass\b)|ure)|ur) ?(argue|look|may|might|must|seem|sound|talk|write)? ?(be|like)? ?(an?|the)? ?(\w+ly|fucking|poor|very)? ?((dumb|jack)?a(rse|ss)\W?(clown|fuck|hat|hole|munch|sex|tard|tastic|wipe)?|bastard|bitch|cunt|delusional|dick|dimwit|disgusting|dolt|douche|dumb|fool(ish)?|fuck(?!ing)\w+|idiot|imbecile|inbred|jerk|loser|moron|naive|narcissist\w*|nitwit|pathetic|queer|schmuck|scum\w*|st[ou]+pid|waste of \w+)(es|s)?', '(yo)?u ?w[0oau]t ?m[8a]t?e?', '(yo)?u can\W*t explain that', '(yo)?u lucky', '(yo)?u monster', '(yo)?u must be new', '(yo)?u tagged as', '(yo)?u wouldn\W*t download an?', '(yo)?u[.,]? I like (yo)?u', '0.?1.?1.?8.?9.?9.?9.?8.?8.?1.?9.?9.?9.?1.?1.?9.?7.?2.?53?', '0bama\w*', '[0o][._][0o]', '[ah]{12,}', '[bg]tfo', '[bm]?wa+h[ah]*', '[dh]urr?', '\d+/10', '\d+[a-z]+\d+me', '\w*pikachu\w*', '\w{3,} as fu[ck]{1,2}', '^((my|o+[uw]+c+h+|o+w+|oo+f+)[ie]*(\b|\s)){2}', '^(\S+ ){1,5}is leaking', '^(\W*(420|69|no*i+c+e+))+\W*$', '^R\.?I\.?P(\b|\.)', '^ay+', '^better call', '^classic', '^good bot', '^that\W*s true\W*$', '^thirded', '^this guy', '^til', '^um+', '^what do(?=\?\s*$)', 'af', 'anall?y?', 'are (yo)?u (fucking sorry|me)', 'ass', 'babby', 'bae(?!\Wsystems)', 'banana.?(4|for).?scale', 'be (un)?attractive.{1,30}be (un)?attractive', 'bitch\w*', 'blackjack and hookers', 'blew', 'bljad', 'boners?', 'book\W?mark(ing|ed)?', 'br(ea|o)k(en?|ing|s)? (both )?(of )?((his|my|your) )?arms', 'br[aou]+h?', 'bs', 'bubbles', 'c(ontrol|trl)\W{1,3}F', 'c[ao]n?stanza', 'came (here )?(\w+ )?to (post|say) (that|this)', 'can confirm(?=\W*($|[,.]|am\b))', 'can\W*t? fap to this', 'cat\W?facts?', 'challenge accepted', 'christ', 'chuck testa', 'claps?', 'comment (apocalypse|graveyard)', 'comment(ed|ing) (to|for)', 'cool (story|theory)', 'd[0o]nger', 'damn onions', 'damn', 'dank(est)?', 'dat \w{3,} doe', 'dat', 'dawg', 'delete facebook', 'dick', 'did nazi that c[ou]m+ing', 'ding', 'dis', 'disappoint\w*', 'doesn\W*t matter\W+had sex', 'doin[\Wg]? god\W*s work', 'don\W*t let your dreams be (dream|meme)s', 'drugs?', 'dumb', 'enough (internet|reddit) for', 'epsteins?', 'erin', 'fails?', 'faith in humanity', 'fam', 'feels good man', 'florida.?man', 'for science', 'front page', 'frozen soap', 'ftfy', 'fuck\w*', 'gave me cancer', 'gg\W?wp', 'gig+it+y', 'got\W*e+m+', 'gotta', 'gud jub', 'harambe', 'hero', 'hit (the )?gym', 'hold my\b.{0,36}\bi\W?m goin.? in', 'holy', 'hot pockets?', 'hunter2', 'i (c|see) wh?[auo]t (yo)?u did th[ea]re?', 'i\W?ll just leave this here', 'idk', 'imh?o', 'in soviet russia', 'intensifies', 'it went ok(ay)?', 'it\W*s happening', 'jesus', 'jimmies', 'just the tip', 'kanye', 'karma', 'kek[ek]*s?', 'kills? (it )?with fire', 'knawledge\w*', 'know that feel', 'kobe(?!.beef)', 'l(?!ull(ed|ing|s)?\b)[ou]l[ol]*([sz]|e?d|ing|no(pe)?|wh?[au]t)*', 'laugh', 'lawyer up', 'le (epic|meme|reddit\w*|sigh|tired|wrong)s?', 'lea?rn (how )?to (code|math)', 'life.{1,15}finds a way', 'like a b[ao]u?[sw]s', 'literally hitler', 'literally', 'lmf?ao+', 'lost it at', 'lpt', 'm[ae]h', 'm\.? night', 'm\We\Wt\Wa\We\Wt\Wa', 'mad bro', 'magnificent bastard', 'manly tears', 'me too thanks', 'mind ?([:=]=?|equals)? ?blown', 'mom\W*s spaghetti', 'monies', 'nailed it', 'national treasure', 'nice try', 'nope', 'not even once', 'nothing is impossible', 'now kiss', 'ol(\W|d?e?)? reddit', 'ontas', 'op pl[sz]', 'op will ?(surely )?deliver', 'papa.?bless', 'pathetic', 'patrick', 'perfect\w* balance\w*', 'phrasing', 'plot twist', 'pls', 'poovey', 'popcorn', 'por.?qu[eé] no l\ws dos', 'prepare (yo)?ur anus', 'preston garvey', 'real human be(an|ing)', 'real(?=[!.\s]*$)', 'right in the feels', 'risky click', 'said no one ever', 'save?(ing|d)? (for ((the )?future|later|when i)|from (cell|i?phone|mobile))', 'science bitch\w*', 'see what (he|she|they|u|you) did there', 'seems legit', 'sex', 'shia', 'shit\w*', 'shots fired', 'should feel bad', 'show myself out', 'sighs?\W+unzips?', 'slaves?', 'slow claps?', 'sneks?', 'sounds(?!.*\bplan\b)', 'step [0-9]: \?{3}', 'stfu', 'stop', 'stopped reading', 'stupid games?\b.{0,16}\bstupid prizes?', 'stupid', 'su(cks?|x)', 'switch\W?[ae]\W?ro+', 'tagged (yo)?u as', 'take my money', 'tee\W?hee', 'tendies?', 'th(anks?|x)( you| u)?[ ,]*(to )?(the )?(ameri\w*|bush|democrats?|dnc|donald|gop|obama\w*|president|republicans?|trump)', 'thanks?,? [0o]bama', 'that escalated quickly', 'tho', 'tifu', 'tips fedora', 'tl\W*dr', 'to the top with (yo)?u', 'trackers?', 'trigger(ed|ing|s)', 'ur?', 'user.?name', 'vidyas?', 'vpn', 'w[io]n\w* the internet', 'wants the d', 'was( ?n[o\W]?t)? disappoint(ed)?', 'watch the world burn', 'we did it,? reddit', 'weed', 'weirdest boner', 'wh?[au]tf? (did I just read|is this I don\W*t even|happened here)', 'who\W*s cutting onions', 'why we can\W*t have nice things', 'will get buried', 'woah', 'would read again', 'would\S* (not )?bang', 'wtf', 'wut', 'xd', 'yesterday\W{1,3}(yo)?u said tomorrow']
```


body#most (regex, full-text): ['[^>]+$']


```yaml
body_shorter_than: 50 ignore_blockquotes: true
action: remove
action_reason: "Short and senseless meme [{{match-body#most}}]"
---
    Donger Prevention This rule targets characters used frequently in donger. This rule will also match on some non-English languages. If mathematical, phonetic, or drawing symbols are common on your subreddit, you will need to modify this rule before using it. Also see Emoji Ban. This rule is still under development and may be buggy. Use at your own risk.
---
body+title (regex, includes): ["(?#IPA Extensions)[\u0250-\u02af]", "(?#Combining Diacritical Marks)[\u0300-\u036f]", "(?#Kannada)[\u0c80-\u0cff]", "(?#Thai)[\u0e00-\u0e7f]", "(?#Tibetan)[\u0f00-\u0fff]", "(?#Katakana)[\u30a1-\u30fa]", "(?#Unified Canadian Aboriginal Syllabics)[\u1400-\u167f]", "(?#Phonetic Extensions)[\u1d00-\u1d7f]", "(?#Mathematical Operators)[\u2200-\u22ff]", "(?#Box Drawing)[\u2500-\u257f]", "(?#Halfwidth and Fullwidth Forms)[\uff00-\uff0c\uff0e-\uffef]+"]
action: filter
action_reason: "Possible donger [{{match}}]"
---
    Title Control These are all rules about submission titles. Require Title Tag Replace the list between the brackets with your own list of required tags. Quotes are required around each individual item due to the use of square brackets.
---
~title: ['[Tag1]', '[Tag2]', '[Tag3]']
action: remove
action_reason: "Title missing required title tag"
comment: |
    Your post has been automatically removed because you did not include one of the required title tags.
    Please read the subreddit rules for more information.
```


--- Reserve Certain Keywords This rule removes posts with titles that include keywords such as "announcement", "mods", "megathread". It also alerts the moderators that someone may be trying to impersonate them. This rule can be copy-pasted directly to your AutoModerator configuration without modification. You can customize the list of keywords as needed.


```yaml
---
title (regex): ['admin(istrator)?s?', 'announcements?', 'mega\W?(post|thread)s?', 'mod(erator)?(\W?post)?s?']
action: remove
action_reason: "Moderator-only title [{{match}}]"
comment: |
    Your post has been automatically removed because you used a keyword which is reserved for the subreddit moderators.
modmail: |
    The above post by /u/{{author}}, with title "{{title}}" was removed because it contained a moderator-only keyword.
    Please investigate and make sure that this action was correct.
```


--- Emoji Ban These rules removes titles with certain non-standard characters such as emoji, miscellaneous symbols, and dingbats. --- # Emoji ban title (regex, includes): ["(?#Zero Width Joiner)[\u200d]", "(?#Box Drawing)[\u2500-\u257f]+", "(?#Miscellaneous Symbols)[\u2600-\u26ff]", "(?#Dingbats)[\u2700-\u27ff]", "(?#Braille)[\u2800-\u28ff]", "(?#!Katakana Letter Tu)[\u30c4]", "(?#Various Emoji)[\U0001F000-\U0001FAFF]"]


```yaml
action: remove
action_reason: "Emoji [{{match}}]"
comment: |
    Your [{{kind}}]({{permalink}} in /r/{{subreddit}}) has been automatically removed because you used an emoji or other symbol.
    Please retry your {{kind}} using text characters only.
```


--- This rule may be modified to be a title+body rule in order to also apply to the text in self posts and comments. Some parts may not work well on science, technology, engineering, or mathematical subreddits, especially if applied to body.

### User Control

These rules help restrict who can post.

Throwaway Account Prevention This rule removes all content from accounts created within the last day. This rule can be copy-pasted directly to your AutoModerator configuration without modification. You can optionally change the required account age; valid units for the age are minutes, hours, days, weeks, months, and years (the word must be plural even if the number given is 1)


```yaml
---
author: account_age: "< 1 days"
action: remove
action_reason: "New user"
---
    Troll Prevention Trolls typically accumulate a fair amount of negative comment karma. This rule removes all content posted by users with less than -50 comment karma. This rule can be copy-pasted directly to your AutoModerator configuration without modification, although you can optionally customize the karma threshold. Note that comment karma for a user is limited to -100, so no user will match if you put -100 or lower in the rule.
---
author: comment_karma: "< -50"
action: remove
action_reason: "Low karma user"
---
    User Bot Ban List Often, it may be advantageous to "bot ban" a troll or spammer rather than ban them. An actual ban simply tells them that it's time to create a new account. With a bot ban, some users won't realize they've been banned. Note that there are two formats, simple and extended. The extended format allows you to keep things clearer and add comments, particularly if you have a lot of users in the list. Replace the list between the brackets with your own list of users to bot ban. Simple format:
---
author: name: [username1, username2, username3]
action: remove
action_reason: "User is banned"
---
    Extended format:
---
author: name:
# comment for a section of users - username1 - username2
# comment for specific user
# comment for a section of users - username3
```



```yaml
action: remove
action_reason: "User is banned"
---
    User Whitelist These rules will approve content from specific users. You can use either or both rules. The first rule auto-approves content at the time a user submits or edits it. This is the only way to auto-approve content by a user with a site-wide shadowban. This will also auto-approve some content that contains a site-wide spammed domain (not most submissions using a URL shortener, though). Note that AutoModerator will never approve content by a shadowbanned user unless the user is specifically mentioned by name or you use a regex check (e.g., name (regex): ['.+']). You also have to uncheck the exclude posts by site-wide banned users from modqueue/unmoderated subreddit setting or AutoModerator will never see the user's content to approve it. The second rule will auto-approve content by a user when it is reported by another user (or a moderator). Replace the lists between the brackets with your own list of users to approve.
---
author: name: [username1, username2, username3]
action: approve
action_reason: "Whitelisted user"
---
author: name: [username1, username2, username3]
reports: 1
action: approve
action_reason: "Approve reported content from whitelisted user"
```


---

### Moderator Alerts

Alert the subreddit moderators when certain things happen.

when certain things happen. Reported Items Notify the moderators if something receives a certain number of reports. It is recommended to set the number of required reports to 2 or 3 for smaller subreddits and 3 to 5 for larger ones. This rule can be copy-pasted directly to your AutoModerator configuration with or without modification.


```yaml
---
reports: 2
action: filter
action_reason: "Multiple reports"
modmail: The above {{kind}} by /u/{{author}} has received multiple reports.
```


Please investigate. --- Topic Alert Alert the moderators if someone posts about a certain topic. Replace the list between the brackets with your own list of keywords. Replace topic in the modmail message accordingly.


```yaml
---
title: [keyword1, keyword2, keyword3]
modmail: The above submission by /u/{{author}}, with title "{{title}}" may be about topic.
```


--- Post Alert Notify the moderators when a submission is made. This may be useful for small or new subreddits with less activity. This rule can be copy/pasted directly to your AutoModerator configuration without modification.


```yaml
---
type: submission
modmail: |
    There is a new post in /r/{{subreddit}}! - Title: {{title}} - User: {{author}}
```


--- Meta Drama Alert Alert the moderators that content on their subreddit has been linked to from elsewhere on Reddit. This rule takes advantage of the existence of /u/TotesMessenger, which detects links to other subreddits and comments to alert users that their content has been linked to from elsewhere on reddit. This rule requires that /u/TotesMessenger not be banned (or bot banned) from your subreddit. If you don't want to remove the /u/TotesMessenger comment, remove the top-level action and action_reason lines.


```yaml
---
author: [TotesMessenger]
body (regex, includes): ['\[(/r/\w+)\] \[(.+)\]\((https?://\w+\.reddit\.com/\S+)\)']
action: remove
action_reason: "Remove {{author}} comment after reporting thread, {{author}} is our friend [{{match-2}}]"
modmail_subject: "Submission linked from {{match-body-2}}"
modmail: |
    The following thread in /r/{{subreddit}} has been linked in {{match-body-2}}: **Original:** [{{title}}]({{permalink}}) **Meta post:** [{{match-body-3}}]({{match-body-4}})
```


--- If you would prefer to be alerted via the moderation queue, use this rule instead:


```yaml
---
author: [TotesMessenger]
body (regex, includes): ['\[(/r/\w+)\] \[(.+)\]\((https?://\w+\.reddit\.com/\S+)\)']
action: remove
action_reason: "Remove {{author}} comment after reporting thread, {{author}} is our friend [{{match-2}}]"
parent_submission: action: report
action_reason: "Submission linked from elsewhere [{{match-body-2}}]"
---
    Report Suspicious Content NSFW and NSFL
---
title+body (regex): ['not.safe.for.(work|life)', 'nsf[wl]']
set_nsfw: true
action: report
action_reason: "Not safe [{{match}}]"
```


---

### Dox Detection and User Safety

These rules detect possibly unsafe posts including posts that may contain personal information.

possibly unsafe posts including posts that may contain personal information. For some of the personal information rules, a modmail is sent including a link to report the user to the Reddit admins. Phone Numbers This will filter most US and non-US phone numbers. There are exceptions for common mismatches, some jokes and references (e.g., Jenny's number), and a few crisis hotlines. If there are specific phone numbers that you would like to whitelist, add them to ~body list inside the brackets, within quotes.


```yaml
---
title+body (regex, includes): ['(?#INT)(\+(?![\s\(]*\d{4})|\b011)[\(\) ._-]{0,3}(9[976]\d|8[987530]\d|6[987]\d|5[90]\d|42\d|3[875]\d|2[98654321]\d|9[8543210]|8[6421]|6[6543210]|5[87654321]|4[987654310]|3[9643210]|2[70]|7|1)\b([\(\) ._-]{0,3}\d){5,14}\b', '(?#NA)\(?\b1?\d{3}[\) ._-]{1,3}\d{3}[ ._-]{1,3}\d{4}\b', '(?#UK)\b(?<!\bu/)(?<!\d\.)0(1\d\d(\s*\d){7}|1\d{3}(\s*\d){6}|1\d1(\s*\d){7}|11\d(\s*\d){7}|2\d(\s*\d){8}|169\s*77(\s*\d){4}|1\d{3}(\s*\d){5}|3\d\d(\s*\d){7}|7\d(\s*\d){8}|8\d\d(\s*\d){6,7})\b']
~body (regex): ['(0118\W+999\W+8[18]1|999\W+119\W+7253)', '(?<=\$)\d+(\.\d\d)?[^\w,.]*[+-][^\w,.]*\d+', '(https?://|www\.)\S*([\(\)._-]{0,3}\d){5}\w*', '000.000.0000', '1024\W+2048', '111.111.1111', '222.222.2222', '281\W+330.8004', '505\W+503.4455', '678.999.8212', '800\W+273.8255', '800\W+799.7233', '999.999.9999', '\d*1\W?2\W?3\W?4\W?5\W?6\W?7\W?8\W?9\d*', '\d{3}\W+555\W\d{4}', '\d{3}\W+867.5309', '\w*\d[\)\s]*=\W*\d\w*']
author: is_contributor: false
action: filter
action_reason: "Phone number detected [{{match}}]"
---
    Email Addresses Replace the list on the ~title+body#whitelist line with any email addresses you need to whitelist. You should only whitelist email addresses that Reddit does not consider personal information. If you have no whitelisted email addresses, remove the ~title+body#whitelist line.
---
title+body (regex): ['(?!(abuse|help|info|no-?reply|phishing|service|spoof|support)\@)[\w!#$%&\x27*+\-./=?\^\x60{|}~]+\@([\w-]{1,64}\.)+([a-z]{2,16}|xn--[a-z0-9-]{1,60})']
```


~title+body#whitelist: [okay.address1@example.com, okay.address2@example.com]


```yaml
action: remove
action_reason: "Email address detected [{{match}}]"
modmail_subject: Doxxing Alert!
modmail: |
    {{permalink}} The above {{kind}} by /u/{{author}} was removed because it contained a possible email address.
    Please investigate immediately.
    If the user is doxxing, [ban them](/r/{{subreddit}}/about/banned) and [report them to the Reddit admins](http://www.reddit.com/message/compose?to=%2Fr%2Freddit.com&subject=Doxxing%20Report:%20%2Fu%2F{{author}}) immediately.
```


--- Credit Card Numbers Credit goes to /u/sexrockandroll and /u/Deimorz for the regex.


```yaml
---
title+body (regex): ['\b(?:4[0-9]{12}(?:[0-9]{3})?|5[12345][0-9]{14}|3[47][0-9]{13}|3(?:0[012345]|[68][0-9])[0-9]{11}|6(?:011|5[0-9]{2})[0-9]{12}|(?:2131|1800|35[0-9]{3})[0-9]{11})\b']
action: remove
action_reason: "Credit card number detected [{{match}}]"
modmail_subject: Doxxing Alert!
modmail: |
    {{permalink}} The above {{kind}} by /u/{{author}} was removed because it contained a possible credit card number.
    Please investigate immediately.
    If the user is doxxing, [ban them](/r/{{subreddit}}/about/banned) and [report them to the Reddit admins](http://www.reddit.com/message/compose?to=%2Fr%2Freddit.com&subject=Doxxing%20Report:%20%2Fu%2F{{author}}) immediately.
---
    IPv4 Addresses
---
title+body (regex): ['\b(?!(?#RANGES)(10\.|172\.(1[6-9]|2\d|3[01])\.|169\.254\.|192\.168\.)|(?#SINGLES)(1\.0\.0\.1|1\.1\.1\.1|1\.2\.3\.4|8\.8\.4\.4|8\.8\.8\.8|9\.9\.9\.9|127\.0\.0\.1|149\.112\.112\.112|208\.67\.220\.220|208\.67\.222\.222)\b)((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)\b']
action: remove
action_reason: "IPv4 address detected [{{match}}]"
modmail_subject: Doxxing Alert!
modmail: |
    {{permalink}} The above {{kind}} by /u/{{author}} was removed because it contained a possible IPv4 address.
    Please investigate immediately.
    If the user is doxxing, [ban them](/r/{{subreddit}}/about/banned) and [report them to the Reddit admins](http://www.reddit.com/message/compose?to=%2Fr%2Freddit.com&subject=Doxxing%20Report:%20%2Fu%2F{{author}}) immediately.
---
    Street Addresses Catches US-style street addresses. Does not catch P.O. boxes. This rule is not recommended for subreddits dedicated to particular areas (like cities) where street addresses for businesses are likely to be common. Add whitelisted addresses to the list on the ~title+body#whitelist line, inside the brackets with each item in single quotes and separated by commas. This rule is still under development and may be buggy. Use at your own risk.
---
title+body (regex): ['\W[A-Za-z]?\d{1,6}[A-Za-z]? (E(\.|ast)?|W(\.|est)?|N(\.|orth)?|S(\.|outh)? )?[\p{Pi}\p{Pf}]?\w+( \w+)?[\p{Pi}\p{Pf}]? (st(reet)?|ave(enue)?|r(oa)?d|dr(ive)?|c(our)?t|blvd|boulevard|lane|ln|highway|hwy|route|rt)']
```


~title+body#whitelist (regex): ['(123 main|221b baker) st(reet)?', '(day|dis[ck]|flash|floppy|gb|gen\W?\d+|hour|inch|kilometer|km|mile|minute|nvme|rpm|sata|second|ssd|tb|week|wheel)s? (\w+ )?drive']


```yaml
action: remove
action_reason: "Street address detected [{{match}}]"
modmail_subject: Doxxing Alert!
modmail: |
    {{permalink}} The above {{kind}} by /u/{{author}} was removed because it contained a possible street address.
    Please investigate immediately.
    If the user is doxxing, [ban them](/r/{{subreddit}}/about/banned) and [report them to the Reddit admins](http://www.reddit.com/message/compose?to=%2Fr%2Freddit.com&subject=Doxxing%20Report:%20%2Fu%2F{{author}}) immediately.
```


--- Disguised Links Detect and filter content where Reddit markup is used to disguise one link as another. This rule can be copy-pasted into your AutoModerator config without modification. The regex used in body+title must be the first regex in the line or it will not work. The regex is written for AutoModerator and will require backreference numbering changes to work elsewhere.


```yaml
---
body+title (regex, includes): ['\[\s*(?:https?://)?(([\w-]{1,64}\.)+[a-z][\w-]{1,63}(?=[\s#&/?\]]))[^\]]*\]\x28\s*https?://(?!((en|home|np|www)\.)?\2[\s#&\x29/?]|[\w.-]+\.gov/|www\.google\.com/url\?\S*\2)[^\x29]*\x29']
action: filter
action_reason: "Possible disguised link, please review [{{match}}]"
---
    Username Mentions Detect and remove comments containing username mention "pings". This rule can be copy-pasted into your AutoModerator config without modification. Reddit only sends a notification for comments containing 3 or fewer usernames so the ~body line exempts comments with more than 3 unique usernames. The regex used in ~body must be the first regex in the line or it will not work.
---
type: comment
body (regex, includes): ['(?<!\bhttps://\w{3}\.reddit\.com/)\bu/([\w-]{3,20})']
~body (regex, includes): ['(?<!\bhttps://\w{3}\.reddit\.com/)\bu/([\w-]{3,20}).*(?<!\bhttps://\w{3}\.reddit\.com/)\bu/(?!\2)([\w-]{3,20}).*(?<!\bhttps://\w{3}\.reddit\.com/)\bu/(?!\2|\3)([\w-]{3,20}).*(?<!\bhttps://\w{3}\.reddit\.com/)\bu/(?!\2|\3|\4)[\w-]{3,20}']
action: remove
action_reason: "Remove username mention [{{match-2}}]"
```


---

### Flair

These rules are related to flair. Set Default Flair for New Users Detects users with no flair and assigns a default flair. Replace user text is here with your desired text for the flair and replace userclassishere with the name of your default flair class. Both items stay within quotes.


```yaml
---
author: ~flair_css_class (regex): ['.+']
set_flair: ["user text is here", "userclassishere"]
---
    Flair Ban If a user keeps setting inappropriate flair, use this rule to "flair ban" them. Replace the list between the brackets in the "name" line with your list of users to flair-ban. Note: this rule will clear their flair every time they participate in the subreddit. It will not prevent them from changing their flair again afterwards (Reddit does not provide any way to do that). This only makes it annoying and time-consuming for the user to maintain an inappropriate flair. (Originally, this was also possible with an empty css_class in the second field of set_flair, but that doesn't work anymore.)
---
author: name: [username1, username2, username3]
flair_text (regex): ['.+']
set_flair: ["", "_"]
overwrite_flair: true
---
    Domain-based Link Flair Set automatic link flair on certain domains. It is highly recommended that the flair class used not appear as a link flair template, so that users cannot use it to mislead other redditors. Replace link text is here with your desired text for the flair and replace linkclassishere with the name of your default flair class. Both items stay within quotes. Replace the list within brackets with your own list of domains to receive automatic link flair.
---
domain: [site1.com, site2.com, site3.com]
set_flair: ["link text is here", "linkclassishere"]
---
    Keyword-based Link Flair Assign link flair based on keywords in post title. Replace link text is here with your desired text for the flair and replace linkclassishere with the name of your default flair class. Both items stay within quotes. Replace the list within brackets with your own list of keywords to receive automatic link flair.
---
title: [keyword1, keyword2, keyword3]
set_flair: ["link text is here", "linkclassishere"]
---
    Default Link Flair Set a default link flair on all link submissions. The use of
priority: -1 ensures that this rule is evaluated last (after any other rule that might assign a different link flair).
```


Replace link text is here with your desired text for the flair and replace linkclassishere with the name of your default flair class. Both items stay within quotes.


```yaml
---
type: link submission
priority: -1
set_flair: ["link text is here", "linkclassishere"]
```


---
