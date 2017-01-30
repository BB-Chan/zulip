# Emoji

Emoji seem like a simple idea, but there's actually a ton of
complexity that goes into an effective emoji implementation.  This
document discusses a number of these issues.

Currently, Zulip uses the Noto (Android) emoji set.  We are
considering adding additional emoji sets as options.

## Emoji codes

The Unicode standard has various ranges of characters set aside for
emoji.  So you can put emoji in your terminal using actual unicode
characters like 😀  and 👍.  If you paste those into Zulip, Zulip will
render them as the corresponding emoji image.

However, the Unicode committee did not standardize on a set of
human-readable names for emoji.  So, for example, when using the
popular `:` based style for entering emoji from the keyboard, we have
to decide whether to use `:angry:` or `:angry_face:` to represent an
angry face.  Different products use different approaches, but for
purposes like emoji pickers or autocomplete, you definitely want to
pick exactly one of these names, since otherwise users will always be
seeing duplicates of a given emoji next to each other.

Picking which emoji name to use is surprisingly complicated!  Zulip
has a nice library, `tools/setup/emoji/emoji_setup_utils.py`, which we
use to make sense of these decisions with a relatively small list of
hand-coded exceptions.

## Tooling

Zulip has a tool, `tools/setup/emoji/build_emoji`, that combines
`emoji-map.json` (an open source mapping of emoji names to short
codes that is fairly liberal about including duplicates) and the
Noto emoji font to extract the emoji that we use in the product.

This tool generates a set of files under `static/generated/emoji` (or
really, it generates the `/srv/zulip-emoji-cache/<sha1>/emoji` tree,
and `static/generated/emoji` is a symlink to that tree; we do this in
order to cache old versions to make provisioning and production
deployments super fast in the common case that we haven't changed the
emoji tooling).

The emoji tree generated by this process contains several import elements:
* `emoji_codes.js`: A set of mappings used by the Zulip frontend to
  understand what unicode emoji exist and what their shortnames are,
  used for autocomplete, emoji pickers, etc.  This has been
  deduplicated using the logic in
  `tools/setup/emoji/emoji_setup_utils.py` to generally only have
  `:angry:` and not also `:angry_face:`, since having both is ugly and
  pointless for purposes like autocomplete and emoji pickers.
* `images/emoji/unicode/*.png`: A farm of emoji
* `images/emoji/*.png`: A farm of symlinks from emoji names to the
  `images/emoji/unicode/` tree.  This is used to serve individual emoji
  images, as well as for the
  [backend markdown processor](markdown.html) to know which emoji
  names exist and what unicode emoji / images they map to.  In this
  tree, we currently include all of the emoji in `emoji-map.json`;
  this means that if you send `:angry_face:`, it won't autocomplete,
  but will still work (but not in previews).
* Some CSS and a PNG for an emoji spritesheet, used in Zulip for emoji
  pickers where we would otherwise need to download over 1000 of
  individual emoji images (which would cause a browser performance
  problem).  We will likely eventually replace the
  `images/emoji/unicode/` tree with using this spritesheet as well.