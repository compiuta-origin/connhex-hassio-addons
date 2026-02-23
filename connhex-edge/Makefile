bump:
ifndef VERSION
	$(error VERSION is required. Use: make bump VERSION=x.y.z)
endif
	@sed 's/^version: .*/version: "$(VERSION)"/' config.yaml > /tmp/config.yaml && mv /tmp/config.yaml config.yaml
	@echo "==> Updated config.yaml to version $(VERSION)"
