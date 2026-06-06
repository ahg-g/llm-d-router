# Prefix Cache Scorer Plugin

**Type:** `prefix-cache-scorer`

Scores candidate endpoints using `PrefixCacheMatchInfo` prepared earlier in the request pipeline.

## What it does

For each candidate endpoint, the scorer reads the `PrefixCacheMatchInfo` attribute and computes a match score.

If `referenceContextBlocks` is configured (`> 0`) and the prompt length (`totalBlocks`) is shorter than this baseline threshold, the match ratio is proportionally scaled down by context length:

```text
score = matchBlocks / referenceContextBlocks
```

Otherwise, for prompts meeting or exceeding the reference context length, the standard match ratio is used:

```text
score = matchBlocks / totalBlocks
```

This produces a normalized score in the range `[0, 1]`:

- higher score: more absolute context (or a larger portion of a massive prompt) is expected to be reusable from cache
- lower score: less prefix cache reuse is expected

If the attribute is missing, has the wrong type, or `totalBlocks` is zero, the endpoint receives score `0`.

## Inputs consumed

This scorer consumes:

- `PrefixCacheMatchInfo`

The attribute is typically produced by the approximate prefix cache data producer before scheduling.

## Configuration

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `prefixMatchInfoProducerName` | string | `""` | Name of the producer generating `PrefixCacheMatchInfo`. |
| `referenceContextBlocks` | integer | `0` | Baseline context length (in blocks) considered highly impactful. Prompts shorter than this have their scores scaled down. |

## Operational notes

- The scorer itself does not hash prompts or maintain cache state.
- It only converts previously prepared prefix match data into endpoint scores.
- To be useful, it should be used together with a data producer that populates `PrefixCacheMatchInfo`.
