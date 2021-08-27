Hints Secrets

- secret - stringdata
```
apiVersion: v1
kind: Secret
metadata:
	name: test
stringData:
		token: Armin ist der beste
```

```
apiVersion: v1
kind: Secret
metadata:
	name: test
data:
		token: QXJtaW4gaXN0IGRlciBiZXN0ZQo=
```