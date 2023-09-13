<script>
export default {
  data() {
    return {
      limit: 10,
      FIRSTNAMES: [
        'John', 'Steve', 'Marc', 'Franklin', 'Isaac', 'Vincent', 'Edwin',
        'Ashlyn', 'Anthony', 'Alia', 'Abby', 'Francesca'
      ],
      LASTNAMES: [
        'Shafer', 'Whitetaker', 'Glenn', 'Stephens', 'Sherman', 'Howard',
        'Kent', 'Clay', 'Beck', 'Simmons', 'Briggs', 'Lee', 'Maxwell'
      ],
      TITLES: [
        'CEO', 'President', 'Vice President', 'Director', 'Manager',
        'Developer'
      ],
      BTN_WIDTH: 4,
    }
  },
  methods: {
    initCopies() {
      let targets = [...document.querySelectorAll('*[data-copy]')];

      targets.reverse().forEach((original) => {
        let amount = parseInt(original.getAttribute('data-copy'));

        for (let i = 0; i < amount; i++) {
          let copy = original.cloneNode(true);

          original.parentNode.insertBefore(copy, original.nextSibling);
        }
      });
    },
    initAutoIncrements() {
      let autos = document.querySelectorAll('.auto-increment');

      autos.forEach((auto, i) => {
        auto.innerHTML = i + 1;
      });
    },
    initAutoFirstnames() {
      let autos = document.querySelectorAll('.auto-firstname');

      autos.forEach((auto) => {
        auto.innerHTML = this.FIRSTNAMES[this.rand(0, this.FIRSTNAMES.length)];
      });
    },
    initAutoLastnames() {
      let autos = document.querySelectorAll('.auto-lastname');

      autos.forEach((auto) => {
        auto.innerHTML = this.LASTNAMES[this.rand(0, this.LASTNAMES.length)];
      });
    },
    initAutoTitles() {
      let autos = document.querySelectorAll('.auto-title');

      autos.forEach((auto) => {
        auto.innerHTML = this.TITLES[this.rand(0, this.TITLES.length)];
      });
    },
    initAutoIntegers() {
      let autos = document.querySelectorAll('.auto-integer');

      autos.forEach((auto) => {
        let min = parseInt(auto.getAttribute('min'));
        let max = parseInt(auto.getAttribute('max'));

        auto.innerHTML = this.rand(min, max);
      });
    },
    initPaginations() {
      let paginations = document.querySelectorAll('.pagination');

      paginations.forEach((pagination) => {
        let table = document.getElementById(pagination.getAttribute('data-table'));

        pagination.setAttribute('data-x', '0');

        if (table !== null && table !== undefined) {
          this.createPagination(pagination, table);

          pagination.closest('.table-container').addEventListener('wheel', (event) => {
            event.preventDefault();

            this.scrollPages(
              event.wheelDelta
                ? event.wheelDelta > 0
                : event.deltaY < 0, pagination,
              table
            );
          });
        }
      });
    },
    createPagination(pagination, table) {
      let limit = parseInt(table.getAttribute("data-limit"));
      let rows = table.querySelectorAll('.table-row:not(.table-heading)');
      let page_count = Math.ceil(rows.length / limit);

      if (isNaN(limit))
        limit = 10;

      rows.forEach((row, index) => {
        if (index >= limit)
          row.style.display = 'none';
      });

      for (let i = 0; i < page_count; i++) {
        let new_button = document.createElement('li');

        new_button.innerHTML = "<span>" + (i + 1) + "</span>";

        if (i === 0)
          new_button.classList.add('active');

        pagination.appendChild(new_button);

        new_button.addEventListener('click', () => {
          this.switchPage(pagination, table, i);
        });
      }

      this.updatePaginationInfos(pagination, table, page_count, 0);
      this.initPaginationExtremes(pagination, table, page_count);
    },
    scrollPages(direction, pagination, table) {
      let last_active = pagination.querySelector('li.active');
      let buttons = pagination.querySelectorAll('li');
      let last_index = Array.from(buttons).indexOf(last_active);
      let scroll_index = null;

      if (direction && last_index > 0) {
        scroll_index = last_index - 1;
      } else if (!direction && last_index < buttons.length - 1) {
        scroll_index = last_index + 1;
      }

      if (scroll_index !== null)
        this.switchPage(pagination, table, scroll_index);
    },
    switchPage(pagination, table, index, bypass = -1) {
      let limit = parseInt(table.getAttribute("data-limit"));
      let rows = table.querySelectorAll('.table-row:not(.table-heading)');
      let buttons = pagination.querySelectorAll('li');
      let last_active = pagination.querySelector('li.active');
      let x_pos = parseInt(pagination.getAttribute('data-x'));
      let x_shift = 0;
      let target_pos = (-index + 2) * this.BTN_WIDTH;
      let page_count = Math.ceil(rows.length / limit);

      if (bypass !== -1) {
        x_shift = (-bypass + 2) * this.BTN_WIDTH;
      } else {
        if (index > 1 && index < buttons.length - 2) {
          x_shift = target_pos;
        } else if (index === 1) {
          x_shift = 0;
        } else if (index === page_count - 2) {
          x_shift = (-page_count + 5) * this.BTN_WIDTH;
        } else {
          x_shift = x_pos;
        }
      }

      rows.forEach((row, row_index) => {
        if (row_index < index * limit || row_index >= (index * limit) + limit) {
          row.style.display = 'none';
        } else {
          row.style.display = 'flex';
          row.style.opacity = '0';

          setTimeout(() => {
            row.style.opacity = '1';
          }, 50);
        }
      });

      last_active.classList.remove('active');
      buttons[index].classList.add('active');
      pagination.style.transform = 'translateX(' + x_shift + 'rem)';
      pagination.setAttribute('data-x', x_shift);

      this.updatePaginationInfos(pagination, table, page_count, index);
      this.updatePaginationProgress(pagination, index, page_count - 1);
    },
    updatePaginationInfos(pagination, table, page_count, index) {
      let info = pagination.closest('.pagination-container').querySelector('.pagination-info');

      if (info === null || info === undefined)
        return;

      let from = 0, to = 0;
      let rows = table.querySelectorAll('.table-row:not(.table-heading)');
      let started = false;

      for (let i = 1; i < rows.length; i++) {
        let display = rows[i - 1].style.display;

        if (display !== 'none' && !started) {
          started = true;
          from = i;
        } else if ((display === 'none' && started) || i == rows.length - 1) {
          to = i == rows.length - 1 ? rows.length : i - 1;
          break;
        }
      }

      info.innerHTML = 'Displaying ' + from + '-' + to + ' on page ' + (index + 1) + '/' + page_count;
    },
    initPaginationExtremes(pagination, table, max) {
      let container = pagination.closest('.pagination-container');
      let left = container.querySelector('.pagination-left');
      let right = container.querySelector('.pagination-right');

      if (left !== null && left !== undefined) {
        left.addEventListener('click', () => {
          this.switchPage(pagination, table, 0, Math.min(2, max - 1));
        });
      }

      if (right !== null && right !== undefined) {
        right.addEventListener('click', () => {
          this.switchPage(pagination, table, max - 1, Math.max(0, max - 3));
        });
      }

      this.initPaginationSteppedExtremes(pagination, table, container, max);
    },
    initPaginationSteppedExtremes(pagination, table, container, max) {
      let left = container.querySelector('.pagination-left-one');
      let right = container.querySelector('.pagination-right-one');

      if (left !== null && left !== undefined) {
        left.addEventListener('click', () => {
          let last_active = pagination.querySelector('li.active');
          let buttons = pagination.querySelectorAll('li');
          let last_index = Array.from(buttons).indexOf(last_active);

          if (last_index > 0)
            this.switchPage(pagination, table, last_index - 1);
        });
      }

      if (right !== null && right !== undefined) {
        right.addEventListener('click', () => {
          let last_active = pagination.querySelector('li.active');
          let buttons = pagination.querySelectorAll('li');
          let last_index = Array.from(buttons).indexOf(last_active);

          if (last_index < max - 1)
            this.switchPage(pagination, table, last_index + 1);
        });
      }
    },
    updatePaginationProgress(pagination, index, total) {
      let dot = pagination.closest('.table-container').querySelector('.progress-point');

      dot.style.left = ((index / total) * 100) + '%';
    },
    initAutoLimits() {
      let table_containers = document.querySelectorAll('.table-container');
      let limit = window.innerHeight / 70;

      table_containers.forEach((container) => {
        let table = container.querySelector('.table');

        container.setAttribute('style', '--data-limit: ' + limit);
        table.setAttribute('data-limit', limit);
      });
    },
    rand(a, b) {
      return Math.floor((Math.random() * b) + a);
    },
  },
  mounted() {
    this.initCopies();
    this.initAutoLimits();
    this.initAutoIncrements();
    this.initAutoFirstnames();
    this.initAutoLastnames();
    this.initAutoTitles();
    this.initAutoIntegers();
    this.initPaginations();
  }
}
</script>

<template>
  <div class="table-container" style="--data-limit: ${limit}">
    <div class="table" id="table-0" data-limit="${limit}">
      <div class="table-row table-heading">
        <div class="table-col">
          #
        </div>
        <div class="table-col">
          First name
        </div>
        <div class="table-col">
          Last name
        </div>
        <div class="table-col">
          Title
        </div>
        <div class="table-col">
          Completed
        </div>
      </div>
      <div class="table-row" data-copy="100">
        <div class="table-col">
          <span class="auto-increment">0</span>
        </div>
        <div class="table-col">
          <span class="auto-firstname"></span>
        </div>
        <div class="table-col">
          <span class="auto-lastname"></span>
        </div>
        <div class="table-col">
          <span class="auto-title"></span>
        </div>
        <div class="table-col">
          <span class="auto-integer" min="0" max="250"></span>
        </div>
      </div>
    </div>

    <div class="pagination-container">
      <div class="d-flex row flex-fill align-items-end justify-content-center">
        <span class="pagination-info"></span>
        <button class="pagination-extreme pagination-left">
          <i class="fa-solid fa-angles-left"></i>
        </button>
        <button class="pagination-extreme pagination-left-one">
          <i class="fa-solid fa-angle-left"></i>
        </button>
        <div class="d-flex flex-fill justify-content-center">
          <div class="pagination-wrapper">
            <ul class="pagination" data-table="table-0"></ul>
          </div>
        </div>
        <button class="pagination-extreme pagination-right-one">
          <i class="fa-solid fa-angle-right"></i>
        </button>
        <button class="pagination-extreme pagination-right">
          <i class="fa-solid fa-angles-right"></i>
        </button>
      </div>
    </div>

    <div class="progress-bar">
      <div class="progress-point"></div>
    </div>
  </div>
</template>

<style scoped></style>
