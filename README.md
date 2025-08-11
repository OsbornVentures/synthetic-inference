 # Synthetic Inference

**Compute on contact.** Embed **computable Gaussian-field models** directly into web data so autonomous agents can **recompute, verify, and negotiate** semantics in **zero‑trust** conditions.

**Spec tag:** `MFP-SD/0.1` · **Namespace prefix:** `mfp:`  
**Pillars:** Machine‑First Physics · Zero‑Trust Recomputation · Synthetic Inference

---

## Why

Today’s structured data (JSON‑LD/Schema.org) is descriptive and persuasion‑biased. **Synthetic Inference** turns metadata into an **operational substrate**: a page declares a field + state; agents **recompute** the declared result and emit a **receipt**. Trust shifts from “source says” → “anyone can recompute.”

---

## Canonical Terms

- **Machine‑First Physics (MFP):** Web resources expose **computable epistemic fields** (default: Gaussian/RBF) alongside claims.  
- **Epistemic Field:** A parameterized field over a defined domain (units, bounds, anchors) that yields a **claim result** when recomputed.  
- **Zero‑Trust Recomputation (ZTR):** Any agent can deterministically recompute the claim from the field **without trusting** the publisher or a third party.  
- **Synthetic Inference (SI):** Higher‑order reasoning that composes many verified fields/receipts instead of relying on opaque priors.  
- **Receipt:** Canonical hash over `{model_spec, state, measurement, result}` proving the agent recomputed what was advertised.  
- **Negotiation:** Agent ↔ resource loop to tighten tolerances, switch kernels, or request more anchors until agreement.

---

## Minimal Spec (draft, normative)

**File:** `/.well-known/ai.json` (SHOULD) or `/ai.json` (MAY)  
**Encoding:** UTF‑8 JSON‑LD · **Canonicalization:** RFC 8785 JCS (**MUST** for receipts)

```json
{
  "@context": [
    "https://schema.org",
    "https://structuredweb.org/mfp/0.1/context.json"
  ],
  "type": "WebContent",
  "mfp:spec": "MFP-SD/0.1",
  "mfp:field": {
    "model": "gaussian",
    "kernel": "rbf",
    "params": { "sigma": 1.0, "lengthscale": 0.25, "dims": 1 },
    "domain": { "units": "Celsius", "min": 0, "max": 100 },
    "anchors": [
      { "x": 0, "y": 0 },
      { "x": 100, "y": 100 }
    ]
  },
  "mfp:measurement": {
    "query": { "x": 37.0 },
    "tolerance": { "abs": 0.1 }
  },
  "mfp:claim": {
    "result": 37.0,
    "interpretation": "temperature_calibration_identity"
  },
  "mfp:receipt": {
    "alg": "sha256-jcs",
    "value": "b2b8ad1f...<truncated_hex>",
    "nonce": "optional-per-request"
  }
}
```

### Normative behaviors

- Publishers **MUST** ensure `mfp:claim.result` is recomputable from `mfp:field` and `mfp:measurement` under declared tolerances.  
- Agents **MUST** JCS‑canonicalize the recomputation input before hashing.  
- On failure, agents **SHOULD** **negotiate**: propose tighter `tolerance`, alternate `kernel`, or additional `anchors`.  
- Multiple fields **MAY** appear in `mfp:fields: []` with a `mfp:graph` for dependencies.

---

## Handshake (wire‑simple)

1. **Advertise** → Agent fetches `ai.json`.  
2. **Request** (optional) → `POST /ai` with desired `measurement`/`tolerance`.  
3. **Recompute** → Agent recomputes `result`.  
4. **Receipt** → Agent returns:
   ```json
   { "mfp:receipt": { "alg": "sha256-jcs", "value": "<hex>", "ok": true } }
   ```
5. **Negotiate** (optional) → Iterate until `ok:true` or abort.

> This is **trust‑minimized recomputation** (not ZK). It’s fast and portable; cryptographic attestation can wrap it later if required.

---

## Design Rationale

- **Gaussian/RBF first:** smooth, differentiable, cheap; ideal for calibration, tolerance bands, interpolation. Other kernels **MAY** be registered.  
- **Receipts over signatures:** Signatures prove origin; **receipts prove coherence**. Use both if needed.  
- **JSON‑LD on purpose:** Coexists with Schema.org; adds **executable semantics** rather than replacing taxonomies.  
- **No JS required:** Fields are data, not scripts. Any client can recompute offline.

---

## Wedge Use‑Cases

- **Unit & tolerance contracts:** Publish exact conversions + bands; agents verify before acting.  
- **Sensor fusion / calibration:** Expose device calibration fields; clients recompute drift; attach receipts to logs.  
- **Policy gates:** Replace “we promise” pages with a field; agents recompute compliance pre‑ingest.

---

## Registries (draft)

- `mfp:kernel` → `rbf`, `poly`, `laplace`, `matern32`, `matern52`  
- `mfp:receipt.alg` → `sha256-jcs`, `blake3-jcs`  
- Units SHOULD follow **UCUM** where applicable.

---

## Quick Verifier (pseudocode)

```python
# inputs: field, measurement, claim
phi = rbf(field.params, field.anchors)        # construct kernel
yhat = phi(measurement.query)                 # recompute
ok = abs(yhat - claim.result) <= measurement.tolerance.abs
receipt = sha256(JCS({"field": field,
                      "measurement": measurement,
                      "result": claim.result}))
return {"ok": ok, "receipt": receipt}
```

---

## Versioning

- **Draft v0.1** – single field/claim; RBF default; receipts mandatory.  
- v0.2 – multi‑field graphs, cross‑resource references, streaming measurements.  
- v1.0 – stabilized registries, formal negotiation verbs, security envelope.

---

## Citation

If you reference this work, cite as:

```
Osborn, D. (2025). Synthetic Inference: Machine‑First Physics for Structured Data and Zero‑Trust Recomputation. Preprint (arXiv pending).
```

BibTeX (stub):

```bibtex
@misc{osborn2025syntheticinference,
  title={Synthetic Inference: Machine-First Physics for Structured Data and Zero-Trust Recomputation},
  author={Osborn, Dekker},
  year={2025},
  note={arXiv preprint, forthcoming}
}
```

---

## License

**CC BY-NC-ND 4.0 + Protocol Addendum** (no monetization, structural repackaging, or derivative protocol use without permission; fair-use interop allowed). A more permissive dual license may be offered for commercial adopters. See `LICENSE`.

---

## Contact

Pre‑arXiv review and implementation questions: open an issue or use the email in `CONTACT`. Demo endpoints and a reference client will be linked here as they go live.

---

**Compute on truth, not vibes.** • **Metadata that does math.** • **Agents verify; servers don’t opine.**
