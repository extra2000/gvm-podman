# Changelog

## [2.0.0](https://github.com/extra2000/gvm-podman/compare/v1.0.1...v2.0.0) (2022-05-17)


### ⚠ BREAKING CHANGES

* **gvmd,gsa:** GVMD now listen to port by default instead of listening to socket file `/run/gvmd/gvmd.sock`

### Features

* **tools:** add `gvm-tools` ([6f54f03](https://github.com/extra2000/gvm-podman/commit/6f54f03f9a7acf3ac8dd62e8e44e7d91598b0001))


### Code Refactoring

* **gsa:** change default HTTP port from 80 to 8080 ([eae1a9d](https://github.com/extra2000/gvm-podman/commit/eae1a9da0369cbc6305ad189a54aa5b84e623742))
* **gvmd-pod:** re-add SSL certs ([2eb4ebc](https://github.com/extra2000/gvm-podman/commit/2eb4ebcd368f55ae594ba65b5be8d6fd9c1b8566))
* **gvmd,gsa:** make GVMD listen to port instead of socket file ([3585757](https://github.com/extra2000/gvm-podman/commit/3585757400b1f5139cd4b987981c518bbffc60c2))


### Documentations

* **deployment:** add instructions to change password ([27b13e6](https://github.com/extra2000/gvm-podman/commit/27b13e68737e97d17b5f0c90bd03efdec45eaa81))
* **deployment:** fix typo for `ospd-openvas` systemd ([8223c03](https://github.com/extra2000/gvm-podman/commit/8223c036a49ac0f8e106b95ac4ae17b94f88d37a))


### Continuous Integrations

* disable test due to broken Kubic mirror ([de50ed2](https://github.com/extra2000/gvm-podman/commit/de50ed2ded0a8d148ca789c1fafb9b977150298a))

### [1.0.1](https://github.com/extra2000/gvm-podman/compare/v1.0.0...v1.0.1) (2022-04-29)


### Code Refactoring

* **gvmd-pod:** remove unused SSL certs ([10e5518](https://github.com/extra2000/gvm-podman/commit/10e5518b5b51ab3a90ae500dc79900cbe9819acd))
* **ospd-openvas-pod:** remove unused SSL certs ([3ff7322](https://github.com/extra2000/gvm-podman/commit/3ff7322cb922f1e5caff1d7727db6293a1192a16))


### Documentations

* **deployments:** fix typo for systemd instructions ([7d664cb](https://github.com/extra2000/gvm-podman/commit/7d664cba3ab61062df640891aa3c0ea7ec6efe86))

## 1.0.0 (2022-04-28)


### Features

* initial implementations ([66427b7](https://github.com/extra2000/gvm-podman/commit/66427b722a920535fe6efa827077c943377d016b))


### Documentations

* **README:** update `README.md` ([dd7b96b](https://github.com/extra2000/gvm-podman/commit/dd7b96bc59676abc4a71fd339bcc7eabe4e8b4ff))


### Continuous Integrations

* add AppVeyor with `semantic-release` ([b8f631a](https://github.com/extra2000/gvm-podman/commit/b8f631ad5f8208cf460c7e0f87eb5dae6fefcf4c))
