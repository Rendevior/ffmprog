#!/bin/sh

# ffmprog
# Easy and Simple progressbar for ffmpeg

trap 'rm ~/.vstats ; save_terminal ; pkill -9 ffmpeg ; printf "\n\n" ; exit 1' INT HUP
[ -e ~/.vstats ] && rm ~/.vstats

save_terminal(){
	printf "\033[?25h"	# Show Cursor
	printf "\033[0m"	# Normal State of Colors
}

sanitize_terminal(){
	printf '%b' "\033[${1}A"
	[ -n "${4}" ] && printf '%b' "\033[${4}B"
	printf "%${2}s" | sed "s_[[:space:]]_\n$(printf "%${termcols}s")_g"
	[ -n "${3}" ] && printf '%b' "\033[${3}A"
}

suc(){
	printf "\033[38;5;2m%b\033[0m\n" "${1}"
}

err(){
	[ -n "${1}" ] && printf "\033[38;5;1m%b\033[0m\n" "${1}" >&2
}

die(){
	err "${2}"
	save_terminal
	exit "${1}"
}

get_infos(){
	if [ -z "${1}" ]; then
		die "1" "No File/URL Returned"
	fi
	inf_meta="$(ffmpeg -i "${1}" 2>&1)"
	vid_dur="$(printf '%s' "${inf_meta}" | sed -n "s/.* Duration: \([^,]*\), start: .*/\1/p")"
	vid_fps="$(printf '%s' "${inf_meta}" | sed -nE '/fps/ {s_.*, (.*) fps.*_\1_p;q}')"
	if [ -z "${vid_dur}" ] || [ -z "${vid_fps}" ]; then
		die "1" "An Error Occured"
	fi
	# Time
	hrs="${vid_dur%%:*}"
	mins="$(printf '%s' "${vid_dur}" | cut -d':' -f2)"
	secs="${vid_dur##*:}" secs="${secs%%.*}" secs="${secs##0}"
	vid_time="$((hrs * 3600 + mins * 60 + secs))"
	total_frames="$(printf '%s\n' "${vid_time} * ${vid_fps}" | bc)" total_frames="${total_frames%%.*}"
}

run_background(){
	ffmpeg -progress ~/.vstats -y "${@}" 2>/dev/null &
	ffmpeg_pid="${!}"
}

progress(){
	printf "\033[?25l"
	while [ -e "/proc/${ffmpeg_pid}" ]; do
		if [ -e ~/.vstats ]; then
			cur_frame="$(grep "frame=" ~/.vstats | tail -n 1)" cur_frame="${cur_frame##*=}"
			cur_bitr="$(grep "bitrate=" ~/.vstats | tail -n 1)" cur_bitr="${cur_bitr##*=}"
			if [ -n "${cur_frame}" ]; then
				progbar "${cur_frame}" "${total_frames}" "Status: ${cur_frame:-0}/${total_frames}"
				printf '\n%s\n%s\n%s\033[3A' "ffmpeg PID: ${ffmpeg_pid}     " "Length: ${vid_dur}     " "Bitrate: ${cur_bitr:-0kbits/s}     "
				sleep 0.3	# Interval to avoid using too much memory (adjust if you want)
			fi
		fi
	done
	save_terminal
	sanitize_terminal 5 4 4 4
	[ "${prog:-0}" -ge 90 ] && suc "\nDownload Complete!!"
	[ -e ~/.vstats ] && rm ~/.vstats
}

progbar(){
	prog="$(( ${1} * 100 / ${2} * 100 / 100 ))"
	done="$(( (prog * 4) / 10 ))"
	left="$(( 40 - done ))"
	fill="$(printf "\033[7m\033[38;5;$(( (done % 103 / 20) + 101 ))m%${done}s\033[0m")"
	empty="$(printf "%${left}s")"
	printf '\033[2K\r%s — %s : %s' "¦${fill}${empty}¦" "${prog}%" "${3}"
	unset prog "done" left fill empty
}

dep_check(){
	for deppack; do
		if ! command -v "${deppack}" >/dev/null ; then
			die "1" "Program \"${deppack}\" is not installed."
		fi
	done
}

# Variable
file_url="$(printf '%s' "${*}" | sed -E 's_.*-i ([^ ]*).*_\1_g')"
termcols="${COLUMNS:-$(stty size)}" termcols="${termcols##* }"

# Call Functions
dep_check "bc" "ffmpeg" "sed" "grep"
get_infos "${file_url}"
run_background "${@}"
progress
