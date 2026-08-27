# Toolchain and repository

C++20, CMake ≥ 3.25, Conan 2, GCC 13 / Clang 17, Ubuntu 24.04 x86-64.

```
swans/
  CMakeLists.txt  conanfile.py  .clang-format  .clang-tidy
  schemas/            SBE xml; FIX data dictionary (custom tags 7928, 20001–20064)
  libs/  core/ (ids, time, journal, pools, fixed-point)  risk/ (libswansrisk)  bus/ (Aeron+SBE)  fixutil/
  services/  refdata/ gateway/ engine/ (with pretrade stage) trades/ positions/ riskengine/ margin/ settlement/ mdpub/ reporting/ api/ rfq/
  tools/     refdata_gen/ ccp_sim/ fix_conformance/ backtest/ replay/
  tests/     gtest; property tests; golden files
  deploy/    docker-compose (prototype); systemd + ansible (prod)
  docs/      this documentation
```

Dependencies: `quickfix`, `aeron`, `eigen`, `spdlog`, `fmt`, `gtest`, `libpqxx`, `nlohmann_json`, `grpc`, `openssl`. Rules: no exceptions or heap allocation in the order path after warm-up; `-Wall -Wextra -Werror`; clang-tidy clean; every message type round-trip tested.
