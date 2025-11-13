# Gutenberg Live Poll Block

A lightweight, extensible, and performance-aware polling block for the WordPress block editor.

The Gutenberg Live Poll Block adds an interactive poll component to the WordPress editor, allowing site owners to create single-choice or multi-choice polls directly within posts and pages. Votes are submitted through the WordPress REST API, stored in a custom database table, and results update in near real-time through efficient client-side polling.

This plugin demonstrates a modern approach to building dynamic, user-interactive features in WordPress using:

- The Block Editor (Gutenberg)

- React-powered block interfaces

- Custom REST API routes

- Secure nonce-based request handling

- Custom database tables for scalable data storage

- Server-side rendering with client-side hydration

It is intentionally implemented without WebSockets to highlight a performant and maintainable “polling-first” architecture that scales gracefully across typical WordPress hosting environments.

# 🚀 Features

- Gutenberg Block – Create polls using a native block with full editor support.

- Multiple Choice Modes – Single-select or multi-select options.

- Custom Table Storage – Votes stored in a dedicated, indexed MySQL table.

- REST-Driven Interaction – Real-time voting and result updates via custom REST routes.

- Live Results – Frontend polls for updated results every few seconds.

- Voter Fingerprint Logic – Soft deduplication of votes without requiring authentication.

- Server-Side Rendered – Ensures consistent markup for SEO and SSR environments.

- Secure – Uses WP nonces, sanitization, and server-side validation.

- Extensible API – Developers can customize intervals, caching, and validation hooks.

# 🧰 Plugin Architecture

This plugin is structured around three pillars:

1. Block Editor (React + Gutenberg)

The editor interface enables users to:

- Enter a question

- Add/edit/remove options

- Choose single or multiple selections

- Configure visibility mode and closing date

Each poll is assigned a stable pollId (UUID) on creation, allowing the backend to map incoming votes to the correct poll instance.

2. Backend (PHP + Custom DB Table + REST API)

A dedicated table wp_glp_votes stores each vote efficiently, with indexes optimized for:

- Fast aggregation (GROUP BY option_id)

- Voter uniqueness checks

- High-volume read patterns

Custom REST routes provide:

- POST /glp/v1/vote – Submit a vote

- GET /glp/v1/results – Fetch aggregated results

Transient-based caching reduces repeated database hits.

3. Frontend (Hydrated React Component)

The block is server-side rendered into a stable markup container.
React hydrates the container and provides:

- UI for selecting options

- Vote submission

- Polling for updated results

Animated transitions from voting to results view

🏗️ File Structure
gutenberg-live-poll-block/
│

├── gutenberg-live-poll-block.php

├── readme.txt

├── includes/

│   ├── class-glp-plugin.php

│   ├── class-glp-install.php

│   ├── class-glp-rest-controller.php

│   ├── class-glp-vote-storage.php

│   └── class-glp-block-registration.php

│

├── src/

│   ├── index.js

│   ├── edit.js

│   ├── save.js

│   └── frontend.js

│

├── build/

│   ├── index.js

│   ├── index.asset.php

│   └── frontend.js

│

├── package.json

├── webpack.config.js

└── README.md

# 📦 Installation
## Requirements

- WordPress 6.0+

- PHP 7.4+

- MySQL 5.7+ / MariaDB 10.3+

- Node.js 16+ (for development build)

# Steps

1. Download the plugin.

2. Upload it to /wp-content/plugins/gutenberg-live-poll-block/.

3. Activate via Plugins → Installed Plugins.

4. The poll block will appear in the Gutenberg block inserter as “Live Poll”.

#⚙️ Database Schema

The plugin creates a custom table:

CREATE TABLE wp_glp_votes (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  poll_id VARCHAR(64) NOT NULL,
  option_id VARCHAR(64) NOT NULL,
  voter_fingerprint VARCHAR(128) NOT NULL,
  user_id BIGINT UNSIGNED NULL,
  created_at DATETIME NOT NULL,
  PRIMARY KEY (id),
  KEY poll_option (poll_id, option_id),
  KEY poll_voter (poll_id, voter_fingerprint)
);


This structure supports:

- High-volume write patterns

- Fast grouping for aggregated results

- Efficient uniqueness checks

- Extensibility for future analytics

# 🔌 REST API
## Submit Vote
POST /wp-json/glp/v1/vote

## Request
{
  "pollId": "poll_123abc",
  "optionIds": ["opt_1"]
}

## Response
{
  "ok": true,
  "pollId": "poll_123abc",
  "counts": {
    "opt_1": 55,
    "opt_2": 12
  },
  "total": 67
}

## Get Results
GET /wp-json/glp/v1/results?pollId=poll_123abc


Used by the frontend to update results at intervals.

# 🔒 Security

- Nonce verification for all write requests

- Sanitized/validated attributes before vote processing

- Voter fingerprint prevents repeated votes:

    - Logged-in users → user ID

    - Anonymous → hashed IP + UA

- Strict validation against block configuration

- Prepared SQL queries for all inserts/selects

This plugin is built according to WordPress security best practices.

# ⚡ Performance Design

The plugin is optimized for high-traffic sites:

## ✔ Custom table

Avoids postmeta bottlenecks for large data.

## ✔ Transient caching

Aggregated results stored temporarily to reduce DB load.

## ✔ Lightweight polling

Polling interval defaults to 5 seconds, adjustable through filters:

add_filter('glp_polling_interval', fn() => 8000); // 8 seconds

## ✔ Single query aggregation

SELECT option_id, COUNT(*) FROM ... GROUP BY option_id

## ✔ Zero WebSockets

Maintains compatibility with shared hosting and avoids over-engineering.

# 🔧 Developer Hooks
## Filters
glp_polling_interval
glp_vote_validation
glp_results_cache_ttl

## Actions
do_action('glp_vote_cast', $pollId, $optionIds, $fingerprint);


These hooks enable full customization of UX, validation, and storage.

# 🧪 Example Usage
## Editor

1. Insert Live Poll block

2. Type your question

3. Add options (e.g. “Yes”, “No”, “Maybe”)

4. Choose single or multi-select

4. Publish the post

## Frontend

- Visitors vote once

- Poll updates results automatically

- Result bars animate smoothly to reflect new totals

# 📚 Why Polling Instead of WebSockets?

This plugin intentionally avoids WebSockets for:

- Compatibility with shared hosting

- Lower operational overhead

- Simplicity and maintainability

- Fewer failure points

- Easy degradation

- Alignment with WordPress’ request-driven architecture

# 🧭 Design Philosophy

The plugin follows these principles:

- Simplicity over premature abstraction

- Clear boundaries between frontend, backend, and storage

- Minimal JS outside React/Gutenberg

- Predictable execution flow

- Graceful failure paths

- Async-friendly interactions

- Performance by default, not as an afterthought

# 🛣 Roadmap (Optional Enhancements)

- Poll analytics dashboard (engagement rate, vote timeline)

- Editor preview mode

- Poll expiration + scheduled closing

- Cookie-based redundancy control

- Server-side setting for default polling interval

- Slack/Discord notifications when poll closes

- AI-generated poll options (OpenAI, optional add-on)

🧑‍💻 Contributing

Pull requests are welcome.
Please ensure:

- Small, focused PRs

- Clear descriptions

- Backward compatibility maintained

- PHPCS/WPCS standards followed

- All new features tested manually

# 📄 License

Released under the GPL-2.0+ license, matching WordPress core’s open-source philosophy.
