#!/usr/bin/env python3
import argparse
import os
import numpy as np
from matplotlib import font_manager, rcParams
import matplotlib.pyplot as plt

PLANS = [
    {"id": "free", "label": "免费 (Free)", "monthly": 0.0, "included": 50, "can_buy_extra": False},
    {"id": "student", "label": "学生 (Student)", "monthly": 0.0, "included": 300, "can_buy_extra": True},
    {"id": "pro", "label": "专业 (Pro $10/月)", "monthly": 10.0, "included": 300, "can_buy_extra": True},
    {"id": "pro_plus", "label": "专业+ (Pro+ $39/月)", "monthly": 39.0, "included": 1500, "can_buy_extra": True},
]

DEFAULT_EXTRA_PRICE = 0.04


def configure_chinese_font():
    preferred_fonts = [
        "Microsoft YaHei",
        "SimHei",
        "PingFang SC",
        "Noto Sans CJK SC",
        "Source Han Sans SC",
        "DengXian",
        "Arial Unicode MS",
    ]
    available_fonts = {font.name for font in font_manager.fontManager.ttflist}
    for font_name in preferred_fonts:
        if font_name in available_fonts:
            rcParams["font.family"] = font_name
            rcParams["axes.unicode_minus"] = False
            return font_name

    rcParams["axes.unicode_minus"] = False
    return None

def compute_costs(usages, plan, extra_price):
    usages = np.array(usages)
    included = plan["included"]
    base = plan["monthly"]
    can_buy = plan["can_buy_extra"]
    costs = np.full_like(usages, np.nan, dtype=float)
    within_mask = usages <= included
    costs[within_mask] = base
    if can_buy:
        extra_mask = usages > included
        costs[extra_mask] = base + (usages[extra_mask] - included) * extra_price
    return costs

def build_plot(max_requests=3000, step=1, extra_price=DEFAULT_EXTRA_PRICE, output="subscription_costs.png", show=False, dpi=150):
    usages = np.arange(0, max_requests + 1, step)
    fig, ax = plt.subplots(figsize=(10, 6))

    # 固定颜色顺序，保证可读性
    color_map = {
        "free": "tab:blue",
        "student": "tab:orange",
        "pro": "tab:green",
        "pro_plus": "tab:red",
    }

    # 绘制曲线（只绘制非 NaN 的连续部分，避免出现不必要的断裂）
    for plan in PLANS:
        costs = compute_costs(usages, plan, extra_price)
        valid = ~np.isnan(costs)
        if not valid.any():
            continue
        # 只绘制从 0 到最后一个有效点的连续段（目前数据结构只会在末尾出现 NaN）
        last = np.where(valid)[0][-1]
        ax.plot(usages[: last + 1], costs[: last + 1], label=f"{plan['label']}（含{plan['included']}次）", color=color_map.get(plan['id'], None), linewidth=2)

    # 设置坐标、网格与标题
    ax.set_xlabel("高级请求用量 (次)")
    ax.set_ylabel("每月最终支付金额 (USD)")
    ax.set_title("不同订阅类型的最终支付成本随高级请求用量的变化")
    ax.grid(True, linestyle="--", alpha=0.5)
    ax.set_xlim(0, max_requests)
    ax.set_ylim(bottom=0)

    # 在每个包含点绘制标记与注释；对无法购买额外请求的订阅区间加阴影提示
    limited_plans = []
    # 先绘制包含点（避免注释位置受后续操作影响）
    for plan in PLANS:
        if plan["included"] <= max_requests:
            included_x = plan["included"]
            included_cost = compute_costs(np.array([included_x]), plan, extra_price)[0]
            if not np.isnan(included_cost):
                ax.scatter([included_x], [included_cost], color=color_map.get(plan['id'], None), s=60, edgecolor='k', zorder=5)
                # 注释偏移根据当前坐标轴范围自动调整，减少重叠
                y_min, y_max = ax.get_ylim()
                rel = (included_cost - y_min) / (y_max - y_min + 1e-9)
                if rel < 0.15:
                    offset = (5, 5)
                elif rel > 0.85:
                    offset = (5, -12)
                else:
                    offset = (5, 5)
                ax.annotate(f"含量={included_x}", (included_x, included_cost), textcoords="offset points", xytext=offset, fontsize=8, color=color_map.get(plan['id'], None))
            if not plan.get("can_buy_extra", True) and plan["included"] < max_requests:
                limited_plans.append(plan)

    # 为所有不可购买额外请求的订阅在图上加阴影并标注说明
    if limited_plans:
        y_min, y_max = ax.get_ylim()
        for plan in limited_plans:
            ax.axvspan(plan["included"], max_requests, color="grey", alpha=0.12, zorder=0)
            ax.text(plan["included"] + max(1, int((max_requests - plan["included"]) * 0.02)), y_max * 0.95, f"超出{plan['label']}不可购买", color="grey", fontsize=9, va='top')

    # 将图例放到图外，避免遮挡曲线
    ax.legend(loc='upper left', bbox_to_anchor=(1.02, 1), borderaxespad=0)
    fig.tight_layout()
    fig.subplots_adjust(right=0.78)

    # 保存高分辨率图片
    fig.savefig(output, dpi=dpi, bbox_inches="tight")
    if show:
        plt.show()
    plt.close(fig)
    return output

def sample_table(usages, extra_price=DEFAULT_EXTRA_PRICE):
    rows = []
    for u in usages:
        row = {"usage": u}
        for plan in PLANS:
            costs = compute_costs(np.array([u]), plan, extra_price)[0]
            row[plan["id"]] = "N/A" if np.isnan(costs) else f"{costs:.2f}"
        rows.append(row)
    return rows

def main():
    configure_chinese_font()
    parser = argparse.ArgumentParser(description="绘制订阅类型与高级请求用量的成本曲线")
    parser.add_argument("--max-requests", type=int, default=3000)
    parser.add_argument("--step", type=int, default=1)
    parser.add_argument("--extra-price", type=float, default=DEFAULT_EXTRA_PRICE)
    parser.add_argument("--output", type=str, default="subscription_costs.png")
    parser.add_argument("--show", action="store_true")
    args = parser.parse_args()
    out = build_plot(max_requests=args.max_requests, step=args.step, extra_price=args.extra_price, output=args.output, show=args.show)
    sample_usages = [0, 50, 300, 500, 1000, 1500, args.max_requests]
    table = sample_table(sample_usages, args.extra_price)
    print("示例用量下每个订阅类型的每月成本：")
    header = ["用量"] + [p["label"] for p in PLANS]
    print("\t".join(header))
    for r in table:
        line = [str(r["usage"]) ] + [r[p["id"]] for p in PLANS]
        print("\t".join(line))
    print(f"图像已保存到: {os.path.abspath(out)}")

if __name__ == "__main__":
    main()
