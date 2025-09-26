Python go-jsonnet bindings
===========================

This directory provides Python bindings (`_gojsonnet`) for the go-jsonnet library.

Building and installing
-----------------------

First build the C-extension and install (into the current directory):

```bash
cd python
python3 setup.py build --build-platlib .
pip install --no-deps --upgrade .
```

Basic usage
-----------

```python
import os
import _gojsonnet

# 1) Define an import callback that returns (full_path, bytes_content)
def import_callback(base_dir, rel_path):
    full = os.path.join(base_dir, rel_path)
    if not os.path.isfile(full):
        raise RuntimeError(f"File not found: {full}")
    with open(full, 'rb') as f:
        return full, f.read()

# 2) Define any native callbacks you need
def concat(a, b):
    return a + b

native_callbacks = {
    'concat': (('a', 'b'), concat),
}

# 3) Load a Jsonnet snippet (here from testdata/basic_check.jsonnet)
snippet_path = os.path.join(os.path.dirname(__file__), 'testdata', 'basic_check.jsonnet')
with open(snippet_path) as fin:
    snippet = fin.read()

# 4) One-off evaluation (parses & executes)
json_str = _gojsonnet.evaluate_snippet(
    'basic_check.jsonnet', snippet,
    import_callback=import_callback,
    native_callbacks=native_callbacks,
)
print(json_str)

# 5) Prepare the snippet once (parse + VM setup)
prepared = _gojsonnet.prepare_snippet(
    'basic_check.jsonnet', snippet,
    import_callback=import_callback,
    native_callbacks=native_callbacks,
)

# 6) Run prepared snippet repeatedly without reparsing
out1 = _gojsonnet.run_prepared(prepared)
out2 = _gojsonnet.run_prepared(prepared)
assert out1 == out2
```

Contents of `testdata/basic_check.jsonnet`:
```jsonnet
std.assertEqual(({ x: 1, y: self.x } { x: 2 }).y, 2) &&
std.assertEqual(std.native("concat")("foo", "bar"), "foobar") &&
std.assertEqual(std.native("return_types")(), {a: [1, 2, 3, null, []], b: 1, c: true, d: null, e: {x: 1, y: 2, z: ["foo"]}}) &&
true
```

See `python/_jsonnet_test.py` for more examples and import/natives implementations.
```