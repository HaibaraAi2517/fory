# Python inspection validation and AI review evidence

Date: 2026-09-27. This record was assembled by the implementation agent. It is not a claim of human verification.

## Reviewed state

- Base: `apache/fory` commit `9043ae6503138ce4ebc28aaf05baed50ccbd7986` (freshly fetched).
- Local reviewed implementation: `6998413b67e30db1fea93ce444552fc4a51d69c7`.
- Published implementation: `HaibaraAi2517/fory` commit `c31f05207f47c4e9c9cb47f22514ed9478b76260`.
- Both Git trees: `146a044e6f425275c7543820a464218900b80363`.
- `git diff --exit-code HEAD origin/feat/python-yaml-inspection` returned 0 after fetching the published branch. Web upload split the implementation across commits; final file contents and modes are identical.

## Fory-guided reviewer result

Fresh clean-context, read-only review following AGENTS.md, AI_POLICY.md and the repository review workflow. Final response:

> Fory-guided review: no actionable findings.
>
> Reviewed the current working diff against `9043ae6503138ce4ebc28aaf05baed50ccbd7986`, including all three untracked implementation/test/documentation files and the tracked packaging, CI, and documentation changes.
>
> The inspection command uses existing strict decoding, preserves display references and map key distinctions, and adds no wire-format or serialization hot-path changes. Documentation accurately bounds the supported scope.
>
> Limitations: this was read-only source review; I ran no tests, builds, or linters. Reported Python/Cython results were supplied by the coordinator, not independently reproduced. Java sample validation was still pending.

The Java sample subsequently passed, as recorded below. No implementation changes followed this review.

## Independent general reviewer result

Separate fresh clean-context, read-only review without loading the Fory-specific review checklist. Final response:

> Independent general review: no further actionable comments.
>
> Reviewed all six changed files in commit `6998413b67e30db1fea93ce444552fc4a51d69c7` against coordinator-fetched `apache/main` baseline `9043ae6503138ce4ebc28aaf05baed50ccbd7986`, including CLI implementation, tests, documentation, dependencies, and CI wiring.
>
> Followed the independent-review carve-out without loading the Fory-specific checklist. Review was read-only; no tests, builds, or linters were run. Runtime validation remains dependent on the coordinator's verification evidence.

## Validation

Windows, CPython 3.13.7, PyYAML 6.0.3, pytest 9.1.1. Editable main source was installed in the test interpreter. Official pyfory 1.7.5 native extensions were reused because a local C++/Bazel toolchain was unavailable. Comparison of tag v1.7.5 to the baseline showed only version changes under Python and C++; native extensions were not rebuilt locally.

From `python/`, with `ENABLE_FORY_DEBUG_OUTPUT=1`, each command was run with `ENABLE_FORY_CYTHON_SERIALIZATION=0` and then `1`:

```text
python -m pytest -q pyfory/tests/test_inspect.py
24 passed (each mode)
```

The focused existing struct, metadata sharing, root-header and reference-tracking regression selection passed 161 tests in each mode. This is focused validation, not the full Python suite or the complete cross-language CI matrix.

```text
ruff check pyfory/inspect.py pyfory/tests/test_inspect.py
All checks passed
ruff format --check pyfory/inspect.py pyfory/tests/test_inspect.py
2 files already formatted
```

Prettier passed for the two changed Markdown documents; `git diff --check` passed.

## Real Java sample

Official Maven artifact `org.apache.fory:fory-core:1.7.5`, JDK 21.0.10. Generator:

```java
import java.nio.file.Files;
import java.nio.file.Path;
import org.apache.fory.Fory;
import org.apache.fory.annotation.ForyStruct;

public class InspectFixture {
  @ForyStruct
  public static class Sample {
    public int count;
    public String label;
  }

  public static void main(String[] args) throws Exception {
    Fory fory = Fory.builder().withXlang(true).withCompatible(true).build();
    fory.register(Sample.class, 901);
    Sample sample = new Sample();
    sample.count = 42;
    sample.label = "java fixture";
    Files.write(Path.of(args[0]), fory.serialize(sample));
  }
}
```

```text
javac -cp fory-core.jar -d . InspectFixture.java
java -cp "fory-core.jar;." InspectFixture compatible-struct.bin
python -m pyfory.inspect compatible-struct.bin
```

Compilation, serialization and both inspection modes exited 0. Both YAML outputs matched and included UnknownStruct user_type_id 901, count 42, label `java fixture`.

## Outstanding contributor work

Human line-by-line review, personally performed verification, provenance confirmation, and human-authored design rationale are pending. AI review is not a substitute for these. The PR must remain Draft until the contributor completes the applicable AI_POLICY.md requirements. No maintainer review is requested by this record.
