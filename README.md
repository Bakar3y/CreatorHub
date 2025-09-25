# CreatorHub

A decentralized digital content creation and royalty distribution platform empowering creators with transparent reward mechanisms.

## Overview

CreatorHub provides a blockchain-based platform for digital content creators to track their contributions and earn royalties through a transparent, automated distribution system. The platform supports various content genres and ensures fair compensation.

## Features

- **Content Genre Certification**: Content curators can certify and approve content genres
- **Creation Activity Tracking**: Record digital content creation with creation credits
- **Royalty Distribution**: Automated royalty reward allocation based on creator contributions
- **Creator Certification**: Complete certification process with proportional royalty rewards

## Smart Contract Functions

### Public Functions
- `establish-creator-platform`: Initialize the digital content creation platform
- `certify-content-genre`: Certify a content genre for tracking
- `record-content-creation`: Record digital content creation activities
- `distribute-royalty-rewards`: Distribute content royalty rewards
- `complete-creator-certification`: Complete certification and claim royalties

### Read-Only Functions
- `get-creator-contributions`: Get creation credits for a creator
- `get-content-genre`: Get content genre for a creator
- `get-total-creation-credits`: Get total creation credits in system
- `is-genre-certified`: Check if content genre is certified

## Getting Started

1. Deploy the contract to Stacks blockchain
2. Establish creator platform with authorized curator
3. Certify content genres for tracking
4. Begin recording digital content creation activities

## License

MIT License
\`\`\`

```clarity file="project-5-innovatechain/contracts/innovatechain.clar"
;; InnovateChain - Research and development innovation rewards platform
(define-data-var research-director principal tx-sender)
(define-data-var total-innovation-points uint u0)
(define-data-var patent-reward-multiplier uint u200) ;; patent rewards per innovation level
(define-data-var last-innovation-review uint u0)

(define-map research-progress principal uint)
(define-map innovation-fields principal (string-utf8 64))
(define-map accredited-fields (string-utf8 64) bool)

;; Error codes
(define-constant err-unauthorized-director (err u7100))
(define-constant err-director-already-designated (err u7101))
(define-constant err-invalid-innovation-points (err u7102))
(define-constant err-no-patent-rewards (err u7103))
(define-constant err-no-research-progress (err u7104))
(define-constant err-invalid-innovation-field (err u7105))
(define-constant err-field-not-accredited (err u7106))

;; Verify director authorization
(define-private (is-research-director (caller principal))
  (begin
    (asserts! (is-eq caller (var-get research-director)) err-unauthorized-director)
    (ok true)))

;; Initialize research and development innovation platform
(define-public (establish-innovation-platform (director principal))
  (begin
    (asserts! (is-none (map-get? research-progress director)) err-director-already-designated)
    (var-set research-director director)
    (ok "InnovateChain research and development platform established")))

;; Accredit innovation field for research tracking
(define-public (accredit-innovation-field (field (string-utf8 64)))
  (begin
    (try! (is-research-director tx-sender))
    (asserts! (> (len field) u0) err-invalid-innovation-field)
    (map-set accredited-fields field true)
    (ok "Innovation field accredited for research tracking")))

;; Record research and development progress
(define-public (record-research-progress (innovation-points uint) (innovation-field (string-utf8 64)))
  (begin
    (asserts! (> innovation-points u0) err-invalid-innovation-points)
    (asserts! (default-to false (map-get? accredited-fields innovation-field)) err-field-not-accredited)
    
    (let ((current-progress (default-to u0 (map-get? research-progress tx-sender))))
      (map-set research-progress tx-sender (+ current-progress innovation-points))
      (map-set innovation-fields tx-sender innovation-field)
      (var-set total-innovation-points (+ (var-get total-innovation-points) innovation-points))
      (ok (+ current-progress innovation-points)))))

;; Review patent reward distribution
(define-public (review-patent-rewards)
  (begin
    (try! (is-research-director tx-sender))
    (let ((current-review (+ (var-get last-innovation-review) u1))
          (total-points (var-get total-innovation-points)))
      (asserts! (> total-points (var-get last-innovation-review)) err-no-patent-rewards)
      
      (let ((patent-reward-pool (* (var-get patent-reward-multiplier) total-points)))
        (var-set last-innovation-review current-review)
        (ok patent-reward-pool)))))

;; Complete innovation certification and claim patent rewards
(define-public (complete-innovation-certification)
  (begin
    (let ((progress-points (default-to u0 (map-get? research-progress tx-sender))))
      (asserts! (> progress-points u0) err-no-research-progress)
      
      (let ((total-points (var-get total-innovation-points))
            (base-patent-rewards (* (var-get patent-reward-multiplier) progress-points))
            (innovation-ratio (/ (* progress-points u100000) total-points)))
        
        (let ((final-patent-rewards (/ (* innovation-ratio base-patent-rewards) u100000)))
          (map-delete research-progress tx-sender)
          (map-delete innovation-fields tx-sender)
          (var-set total-innovation-points (- (var-get total-innovation-points) progress-points))
          (ok (+ progress-points final-patent-rewards)))))))

;; Read-only functions
(define-read-only (get-research-progress (researcher principal))
  (default-to u0 (map-get? research-progress researcher)))

(define-read-only (get-innovation-field (researcher principal))
  (map-get? innovation-fields researcher))

(define-read-only (get-total-innovation-points)
  (var-get total-innovation-points))

(define-read-only (is-field-accredited (field (string-utf8 64)))
  (default-to false (map-get? accredited-fields field)))
