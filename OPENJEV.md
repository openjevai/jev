# OpenJEV support

This fork adds optional [OpenJEV](https://openjev.sh) support alongside the
original [TypeSafe](https://typesafe.ai) API. TypeSafe remains the default;
anyone with a TypeSafe key sees zero behaviour change.

## What was added

- `option.go` — new constants (`openjevBaseURL`, `openjevModel`,
  `providerTypeSafe`, `providerOpenJEV`, `envProvider`, `envOpenJEVKey`),
  a `provider` field on `config`, and a `WithProvider` option. The `resolve`
  method now selects the provider before resolving the key, base URL, and
  model, so OpenJEV defaults apply only when OpenJEV is selected.
- `client.go` — a `Provider()` method that returns the selected provider.
- `doc.go` — package docs mention `WithProvider` and OpenJEV.
- `README.md` — a note after the intro and a `WithProvider` row in the
  configuration table.

No TypeSafe code path was renamed, removed, or re-defaulted.

## Provider selection rule

1. Explicit choice wins: `WithProvider("openjev")` or `JEV_PROVIDER=openjev`.
2. Otherwise, if `TYPESAFE_API_KEY` is set → TypeSafe (unchanged default).
3. Otherwise, if only `OPENJEV_API_KEY` is set → OpenJEV.
4. Otherwise → TypeSafe (the original default).

When OpenJEV is selected and key / base URL / model are not set explicitly:

| Setting | OpenJEV default |
| --- | --- |
| API key env | `OPENJEV_API_KEY` |
| Base URL | `https://api.openjev.sh` |
| Model | `openjev` |

`TYPESAFE_BASE_URL` and `TYPESAFE_DEFAULT_MODEL` still override the base URL
and model for either provider.

## How to configure

```sh
# TypeSafe (default, unchanged):
TYPESAFE_API_KEY=... go run ./examples/quickstart

# OpenJEV (explicit):
JEV_PROVIDER=openjev OPENJEV_API_KEY=... go run ./examples/quickstart

# OpenJEV (auto-detected when only OPENJEV_API_KEY is set):
OPENJEV_API_KEY=... go run ./examples/quickstart
```

Or in code:

```go
client, err := jev.New(jev.WithProvider("openjev"))
```

## Verification

- A live `POST https://api.openjev.sh/v1/systemone` request with model
  `openjev`, state `ping`, and one noul question returned HTTP 200.
- `grep -rn 'api\.typesafe.ai'` confirms no hardcoded TypeSafe default was
  introduced or changed; the original `defaultBaseURL` constant is untouched.
- The default retry policy already covers HTTP 500–599, which includes 503
  (OpenJEV overload) and 529 (TypeSafe overload); no retry change was needed.

## Upstream

Original project: https://github.com/kataras/jev by @kataras
