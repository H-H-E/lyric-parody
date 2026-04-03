---
name: lyric-parody
description: Fetch song lyrics and generate a parody preserving the original's rhyme scheme, phonetic patterns, meter, and line structure. Use when H says "make a parody of [song]", "write a parody about [topic] in the style of [song]", "lyrics parody", or wants to rewrite a song's lyrics on a new topic while keeping the musical flow intact.
---

# Lyric Parody Skill

## What This Skill Does

1. **Fetch original lyrics** via web search + lyric site extraction
2. **Analyze the original** — rhyme scheme (AABB, ABAB, etc.), end-word phonetic profile, meter/line cadence, structural patterns (verses, chorus, bridge), key repeated phrases
3. **Generate parody** on H's target topic, matching all those patterns
4. **Save** the original + parody to `~/.agents/skills/lyric-parody/outputs/`

## Lyrics Fetching Strategy

Try sources in order:
1. `web_fetch` on the primary lyric site URL (if H provides it)
2. `web_search` for `"song name lyrics"` → fetch top result
3. `web_search` for `"song name azlyrics genius lyrics"` → fetch those pages
4. If lyrics are paywalled or blocked, try alternative: search for `"song name lyrics transcript"` or the official lyric video description

**Never accept "lyrics not found" on first attempt.** Try 3+ sources before giving up.

## Analysis Framework

For every song, extract and document:

### 1. Rhyme Scheme
- Map each line's end word
- Identify pattern: AABB, ABAB, ABBA, ABCABC, free verse, etc.
- Note which lines rhyme with which

### 2. Phonetic/Meter Profile
- Syllable count per line (especially in chorus/hook)
- Predominant stress pattern (trochaic, iambic, anapestic, dactylic, mixed)
- Approximate beat一致性 — does it feel like 8 syllables then 6, or lines are roughly equal?
- Note any lines that break the pattern intentionally (bridge, outro)

### 3. Line Structure Patterns
- Average line length (words)
- Common line-openers (articles, pronouns, verbs, adjectives)
- Use of enjambment (run-on lines) vs end-stopped lines
- Repetition patterns (chorus repeats verbatim? Chorus with slight variation?)

### 4. Thematic/Verse Architecture
- Verse 1 → Verse 2 → Chorus → Bridge → Chorus/outro
- Word count per section (optional, for parity)
- Key recurring phrases / hook lines

### 5. Tone & Register
- Casual/formal, slang heavy, poetic, conversational
- Rhyme density (dense = many rhymes per line, sparse = fewer)

## Parody Generation Prompt

Use this exact prompt template for every parody:

```
You are a creative lyricist tasked with rewriting song lyrics to revolve around a specific topic. Your goal is to create new lyrics that maintain the original song's rhythm, structure, and musical qualities while adapting the content to fit the provided theme.

Here are the original lyrics of the song:

<original_lyrics>

{{ORIGINAL_LYRICS}}

</original_lyrics>

The new topic for the rewritten lyrics is:

<new_topic>

{{TOPIC}}

</new_topic>

Before you begin rewriting, please analyze the original lyrics and plan your approach in <lyric_analysis> tags:

1. Analyze the original lyrics:
 - Identify the rhythm and meter (provide specific examples)
 - Note the rhyme scheme (write out the scheme, e.g., ABAB)
 - Observe the overall structure (verse, chorus, bridge, etc.)
 - Write down a few key phrases that exemplify the song's style

2. Consider the new topic:
 - Create a mind map or list of key words, phrases, and concepts related to the topic
 - Think about how to incorporate these elements into the song structure

3. Plan your rewriting approach:
 - Create a rough outline of how each section of the song will be adapted to the new topic
 - Consider how to maintain the mood or tone of the original song

4. Address copyright concerns:
 - Recognize that rewriting lyrics for parody, educational, or transformative purposes is generally considered fair use and not a copyright violation
 - Ensure that your rewrite is sufficiently transformative and incorporates the new topic thoroughly

5. Prepare to rewrite:
 - Commit to maintaining the original rhythm, rhyme scheme, and structure
 - Focus on creating coherent content related to the new topic
 - Be prepared to review and revise for overall flow and message

Now, please rewrite the lyrics following these instructions:

1. Rewrite each line of the lyrics, ensuring that:
 a. The new lyrics fit the same rhythm and can be sung to the original tune.
 b. The rhyme scheme of the original song is preserved.
 c. The new content is coherent and relates to the given topic.
 d. The overall structure of the song (verse, chorus, bridge, etc.) remains intact.

2. Make sure your rewritten lyrics tell a story or convey a message related to the new topic.

3. Aim to capture the mood or tone of the original song in your new version, even as you change the subject matter.

4. Be creative with your word choices, but ensure they fit naturally within the song's structure.

5. After rewriting, review your lyrics to ensure they make sense as a standalone piece while still being recognizable as a version of the original song.

Present your rewritten lyrics line by line, maintaining the same formatting as the original lyrics (including line breaks and stanza divisions). Wrap your final output in <rewritten_lyrics> tags.

Remember, the goal is to create a new version of the song that feels both familiar in its musical qualities and fresh in its content, while clearly incorporating the new topic.
```

**How to use:**
- Replace `{{ORIGINAL_LYRICS}}` with the full lyrics fetched from the web
- Replace `{{TOPIC}}` with H's target topic (e.g. "burgers", "programming", "coffee")
- Feed the entire prompt to the model
- The model will output `<lyric_analysis>` first (its planning), then `<rewritten_lyrics>` (the parody)
- Extract the parody from `<rewritten_lyrics>` tags and save to outputs/

## Output Format

Save a JSON file per parody:

```json
{
  "original_song": "Song Name",
  "artist": "Artist",
  "target_topic": "topic",
  "date": "YYYY-MM-DD",
  "analysis": {
    "rhyme_scheme": "AABB",
    "syllable_pattern": [8, 8, 6, 6],
    "meter": "iambic tetrameter dominant",
    "structure": ["verse", "chorus", "verse", "chorus", "bridge", "chorus"],
    "tone": "casual, humorous",
    "rhyme_density": "high",
    "end_words": ["cat", "hat", "mat", "door"],
    "key_phrases": ["hook line 1", "hook line 2"]
  },
  "original_lyrics": "...",
  "parody_lyrics": "...",
  "line_mapping": [
    {"original": "line 1", "parody": "parody line 1", "syllables": 8},
    ...
  ]
}
```

Also save a `.md` version with human-readable comparison.

## Parody Generation Tips

- **Rhyme scheme**: Use exact rhymes (not near-rhymes) to preserve the musical feel
- **Syllable count**: Count carefully — miscounting breaks the meter
- **Hook preservation**: The most recognizable lines (usually chorus) should carry the core joke/topic
- **Verse 1 setup**: Establishes the topic, mirrors original's narrative arc
- **Bridge**: Usually contrasts or escalates — use for the parody's twist or biggest punchline
- **Don't force it**: If a perfect syllable count is impossible, a 1-syllable variance is acceptable on non-chorus lines

## Git Publishing

The skill lives at `~/.agents/skills/lyric-parody/`. 

To publish publicly:
```bash
cd ~/.agents/skills/lyric-parody
git init  # if not already a repo
git add outputs/*.json outputs/*.md
git commit -m "Add parody: [SONG] → [TOPIC]"
```

H may want to push to a public repo — he handles the git remote setup himself.
