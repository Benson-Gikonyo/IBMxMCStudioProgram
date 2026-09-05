# Vizier submission

Vizier is an Ethiopian personal-finance intelligence prototype for IBM Dev Day. The application helps individuals organize monthly finances, import statements, understand financial health, track goals and assets, and ask an educational AI coach questions grounded in deterministic calculations.

## Product boundary

Vizier is not a payment processor, lender, bank, broker or regulatory-reporting platform. It does not connect to telebirr, move money, execute trades, collect national identity numbers or submit reports to the National Bank of Ethiopia.

The accompanying launch and vulnerability files describe controls that would apply if the product later crossed into regulated payments or lending. They are planning references, not claims about the current prototype.

## IBM capability

The AI workspace supports IBM watsonx.ai with an IBM Granite model. Credentials remain optional so the demo can run offline. Vizier calculates financial metrics locally and sends only the assembled context for a question to the configured model.

## Contents

- `vizier-app/`: runnable application, tests, sample Ethiopian statement and deployment files
- `Vizier-Ethiopia-Presentation.pptx`: editable presentation
- `Vizier-Ethiopia-Presentation.pdf`: presentation preview
- `README-checklist.md`: supplied launch-planning reference
- `README-vulnerability.md`: supplied security-assessment reference

## Demo

1. Start the application using `vizier-app/README.md`.
2. Register with a synthetic email and password.
3. Enter an ETB monthly plan.
4. Import `vizier-app/sample-data/ethiopia-statement.csv`.
5. Review the health score, budgets and history.
6. Add a demo Ethiopian asset.
7. Ask the AI workspace about spending or a savings goal.

Do not use real personal, banking or identity data during the demo.
